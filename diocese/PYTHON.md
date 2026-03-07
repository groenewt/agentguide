# Advanced Python kernel-level performance optimization

**Python's default execution model trades raw throughput for developer velocity — but its runtime is far more malleable than most engineers realize.** This reference covers the specific mechanisms by which CPython manages memory, dispatches calls, and schedules threads at the C level, along with concrete techniques to subvert or bypass each bottleneck. Every optimization here targets Python 3.11–3.13+ and assumes familiarity with systems programming concepts. The payoff is substantial: properly applied, these techniques routinely deliver **50–2000× speedups** on compute-bound workloads without leaving the Python ecosystem.

---

## 1. CPython's object model costs 16 bytes before your data begins

Every CPython object starts with a `PyObject` header:

```c
typedef struct _object {
    Py_ssize_t ob_refcnt;    // 8 bytes — reference count
    PyTypeObject *ob_type;   // 8 bytes — pointer to type object
} PyObject;                  // Total: 16 bytes on 64-bit

typedef struct {
    PyObject ob_base;        // 16 bytes
    Py_ssize_t ob_size;      // 8 bytes — element count
} PyVarObject;               // Total: 24 bytes
```

A Python `float` occupies **24 bytes** (16-byte header + 8-byte `double`). A Python `int` starts at **28 bytes**. An empty `dict` costs **64 bytes**. The `sys.getsizeof()` function reports shallow size only — a list reports its struct plus pointer array, not the contained objects:

```python
import sys
sys.getsizeof(3.14)    # 24 bytes (PyFloatObject)
sys.getsizeof(0)       # 28 bytes (PyLongObject, compact)
sys.getsizeof([])      # 56 bytes (PyListObject, empty)
sys.getsizeof({})      # 64 bytes (PyDictObject, empty)
```

The **free-threaded build** (Python 3.13t+) doubles the header to **32 bytes** per object, adding `ob_tid` (owning thread ID), `ob_mutex`, `ob_gc_bits`, and split reference counts (`ob_ref_local` + `ob_ref_shared`). This is the fundamental cost of removing the GIL.

### Reference counting and the generational collector

`Py_INCREF`/`Py_DECREF` are non-atomic increments/decrements guarded by the GIL. When `ob_refcnt` hits zero, `tp_dealloc` fires immediately — deterministic, zero-latency reclamation that handles ~95% of objects. Reference cycles require CPython's **three-generation collector**:

```python
import gc
gc.get_threshold()  # (700, 10, 10) — gen0 every 700 net allocs,
                    # gen1 every 10 gen0 collections, gen2 every 10 gen1
```

Python 3.12+ uses **incremental collection** for the oldest generation, reducing worst-case pause times. For latency-sensitive code, disabling the GC is viable when you guarantee no cycles are created — Instagram's approach saves ~10% memory in forked worker processes by preventing copy-on-write page dirtying from refcount mutations:

```python
gc.disable()
# ... allocation-heavy, cycle-free hot path ...
gc.collect()  # periodic manual sweep if needed
```

### pymalloc: arenas, pools, and blocks

Objects ≤512 bytes bypass `malloc()` entirely through pymalloc's three-tier allocator:

| Level | Size | Details |
|-------|------|---------|
| **Arena** | 256 KiB (1 MiB on 64-bit, 3.11+) | Allocated via `mmap()`, freed only when all pools empty |
| **Pool** | 4 KiB (system page) | One pool per size class, doubly-linked in `usedpools[]` |
| **Block** | 8–512 bytes (multiples of 8) | 64 size classes; O(1) alloc/free via per-class free list |

Requests above 512 bytes fall through to `malloc()`. Inspect with `sys._debugmallocstats()` or set `PYTHONMALLOC=debug`. The free-threaded build replaces pymalloc with **mimalloc**, which provides per-thread heaps essential for lock-free allocation.

### `__slots__` eliminates per-instance dictionaries

Without `__slots__`, each instance carries a `__dict__` costing ~104–296 bytes. With `__slots__`, attributes are stored as a fixed-size `PyObject*` array in the instance struct:

```python
import sys, tracemalloc

class Regular:
    def __init__(self, x, y, z):
        self.x = x; self.y = y; self.z = z

class Slotted:
    __slots__ = ('x', 'y', 'z')
    def __init__(self, x, y, z):
        self.x = x; self.y = y; self.z = z

# Regular: ~48 bytes instance + ~184 bytes __dict__ ≈ 232 bytes
# Slotted: ~64 bytes total — 40-60% savings at scale
# Python 3.10+ supports @dataclass(slots=True)
```

### `struct` for packed binary data and `memoryview` for zero-copy

The `struct` module packs Python values into **compact binary representations** — a 3D point as `struct.pack('ddd', x, y, z)` occupies 24 bytes versus ~136 bytes as a tuple of floats. For zero-copy slicing, `memoryview` wraps any buffer-protocol object without copying:

```python
data = bytearray(1_000_000)
mv = memoryview(data)
slice_view = mv[100:200]     # Zero-copy: pointer + offset only
slice_view[0] = 42
assert data[100] == 42       # Same underlying memory

# Reinterpret bytes as int32 — zero-copy
mv_int32 = mv.cast('i')     # 1M bytes → 250K int32s

# Python 3.12+ (PEP 688): implement __buffer__ in pure Python
class MyBuffer:
    def __init__(self, data: bytes):
        self._data = bytearray(data)
    def __buffer__(self, flags: int) -> memoryview:
        return memoryview(self._data)
```

---

## 2. The GIL protects CPython internals, not your code

The GIL ensures only one thread executes Python bytecode at a time. It protects **reference count integrity** (non-atomic `ob_refcnt++/--`), CPython's internal data structures (dict, list, allocator state), and C extension assumptions about single-threaded access. It does **not** make your Python code thread-safe — `counter += 1` is three bytecodes (`LOAD_GLOBAL`, `BINARY_ADD`, `STORE_GLOBAL`) with GIL releases possible between any of them.

The GIL is released during I/O operations, `time.sleep()`, and by C extensions that explicitly call `Py_BEGIN_ALLOW_THREADS`. NumPy releases it for large array operations. The interpreter yields it every **5ms** (configurable via `sys.setswitchinterval()`).

### Subinterpreters with per-interpreter GIL (3.12+)

PEP 684 introduced per-interpreter GILs. Each subinterpreter can own its own GIL, enabling true parallelism without separate processes:

```python
import _interpreters as interpreters  # Python 3.13+, experimental
import threading

def run_in_subinterp():
    interp = interpreters.create()
    interp.exec('result = sum(range(10_000_000))')
    interp.close()

# Each thread runs a subinterpreter with its own GIL
threads = [threading.Thread(target=run_in_subinterp) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()
```

The tradeoff: subinterpreters cannot share Python objects (data must be serialized), and some C extensions with global state are incompatible.

### Free-threaded Python: PEP 703 reaches supported status

Python 3.13 shipped free-threaded builds as experimental (`python3.13t`). **PEP 779** (accepted June 2025) elevated it to "supported but not default" in Python 3.14. The single-threaded performance gap narrowed from **~40% slower** in 3.13t to **<10% on most platforms** in 3.14t.

Three techniques replace the GIL's refcount protection. **Biased reference counting** gives each object a fast non-atomic `ob_ref_local` for the owning thread and an atomic `ob_ref_shared` for others. **Immortal objects** (PEP 683) make `Py_INCREF`/`Py_DECREF` no-ops for `None`, `True`, small integers, and interned strings, eliminating cache-line contention on the most shared objects. **Deferred reference counting** skips refcounting for long-lived objects like module globals, with the GC periodically scanning thread stacks to account for them.

```python
import sys
sys.get_int_max_str_digits()   # verify 3.13+
print(sys._is_gil_enabled())   # False in free-threaded build

# True parallelism with threads
import threading, time
def countdown(n):
    while n > 0: n -= 1

start = time.time()
t1 = threading.Thread(target=countdown, args=(25_000_000,))
t2 = threading.Thread(target=countdown, args=(25_000_000,))
t1.start(); t2.start(); t1.join(); t2.join()
# Standard build: ~same as single-threaded (GIL serializes)
# Free-threaded: ~half the time (true parallelism)
```

Extension authors must use `Py_BEGIN_CRITICAL_SECTION`/`Py_END_CRITICAL_SECTION` for object access and declare support via `Py_MOD_GIL_NOT_USED`. The ecosystem tracker at py-free-threading.github.io monitors compatibility — NumPy and scikit-learn already have substantial support.

---

## 3. Four JIT strategies with radically different trade-offs

### Numba: LLVM-compiled numeric kernels

Numba translates a Python/NumPy subset to machine code via LLVM. The `@njit` decorator (nopython mode, now the default) performs type inference at first call, compiles to native code, and caches the result. Typical speedups are **50–200× for numeric loops**:

```python
from numba import njit, prange
import numpy as np

@njit(parallel=True, fastmath=True)
def heat_diffusion_step(grid, alpha, dx, dt):
    rows, cols = grid.shape
    new_grid = np.empty_like(grid)
    r = alpha * dt / (dx * dx)
    for i in prange(1, rows - 1):        # OpenMP-style parallelism
        for j in range(1, cols - 1):
            new_grid[i, j] = grid[i, j] + r * (
                grid[i+1,j] + grid[i-1,j] +
                grid[i,j+1] + grid[i,j-1] - 4*grid[i,j])
    return new_grid
```

Numba's `@vectorize` and `@guvectorize` create true NumPy ufuncs compiled to native code, with `target='parallel'` for multi-threading and `target='cuda'` for GPU execution. The key limitation: Numba cannot compile arbitrary Python — no custom classes (only `@jitclass`), no `**kwargs`, limited exception handling.

### CPython 3.13+ copy-and-patch JIT — promising but immature

Python 3.13 introduced a **copy-and-patch JIT** (PEP 744) that builds on the specializing adaptive interpreter from 3.11 (which already delivered ~25% speedup). The pipeline: bytecode → specialized Tier 1 → Tier 2 micro-ops → optimized micro-ops → native code via pre-compiled stencils patched with runtime values.

An honest assessment from Ken Jin (CPython JIT developer, July 2025): the 3.13 JIT "ranges from slower than the interpreter to roughly equivalent." The widely cited "2–9% faster" compared against the Tier 2 *interpreter* (itself slower than Tier 1), not the baseline. **Python 3.14** shows meaningful gains on select workloads (e.g., ~15% on Richards) but is still ~6–13% *slower* on others (nbody, spectral_norm). Enable with `--enable-experimental-jit` at build time or `PYTHON_JIT=1` at runtime. Microsoft's layoffs of the Faster CPython team in May 2025 shifted development to the community.

### Cython typed memoryviews: compiled C with nogil

Cython compiles Python-like code to C, with typed memoryviews providing GIL-free access to buffer protocol objects. Performance on array operations reaches **1,800× faster** than pure Python:

```cython
# heat_kernel.pyx
cimport cython
from cython.parallel import prange

@cython.boundscheck(False)
@cython.wraparound(False)
cpdef double[:, :] heat_step(double[:, ::1] grid, double alpha,
                              double dx, double dt):
    cdef Py_ssize_t i, j, rows = grid.shape[0], cols = grid.shape[1]
    cdef double r = alpha * dt / (dx * dx)
    result_np = np.empty((rows, cols), dtype=np.float64)
    cdef double[:, ::1] result = result_np
    with nogil:  # Releases GIL — enables true multi-threading
        for i in prange(1, rows - 1):
            for j in range(1, cols - 1):
                result[i,j] = grid[i,j] + r * (
                    grid[i+1,j] + grid[i-1,j] +
                    grid[i,j+1] + grid[i,j-1] - 4.0*grid[i,j])
    return result
```

### PyPy: best for pure-Python servers

PyPy's meta-tracing JIT records hot loop traces and compiles them to machine code, averaging **~4.7× faster** than CPython on pure Python benchmarks. It struggles with C extension compatibility and startup time. PyPy 7.3.20 (July 2025) ships Python 3.11 support; no 3.12+ yet.

---

## 4. C extensions: ctypes, cffi, pybind11, and vectorcall

### Choosing the right FFI

For a shared C library (`libmathlib.so` exporting `fib()` and `vector_norm()`):

**ctypes** — stdlib, no compilation, but every argument is marshaled through Python's FFI layer (~1–2μs per call):

```python
import ctypes
lib = ctypes.CDLL("./libmathlib.so")
lib.fib.argtypes = [ctypes.c_ulong]
lib.fib.restype = ctypes.c_ulong
lib.fib(35)  # 9227465
```

**cffi** — cleaner API, two modes. ABI mode (`dlopen`) works like ctypes; API mode (`set_source`) compiles a wrapper with near-native call overhead:

```python
import cffi
ffi = cffi.FFI()
ffi.cdef("unsigned long fib(unsigned long n);")
lib = ffi.dlopen("./libmathlib.so")
lib.fib(35)
```

**pybind11** — header-only C++11 with automatic type conversion and lowest per-call overhead (~0.1–0.5μs):

```cpp
#include <pybind11/pybind11.h>
PYBIND11_MODULE(mathlib, m) {
    m.def("fib", &fib, "Fibonacci", pybind11::arg("n"));
}
```

For compute-heavy functions all three converge since C computation dominates. The difference shows on **many small calls** — pybind11 wins there.

### Releasing the GIL from C extensions

```c
static PyObject* py_heavy_compute(PyObject *self, PyObject *args) {
    // ... extract C data while holding GIL ...
    double result;
    Py_BEGIN_ALLOW_THREADS          // Releases GIL
    result = heavy_compute(data, n); // Pure C — no Python API calls!
    Py_END_ALLOW_THREADS            // Reacquires GIL
    return PyFloat_FromDouble(result);
}
```

In pybind11: `py::gil_scoped_release release;` achieves the same via RAII.

### Vectorcall (PEP 590) eliminates per-call allocations

The legacy `tp_call` convention creates a `tuple` for positional args and a `dict` for kwargs on every call. **Vectorcall** passes a flat C array of `PyObject*` pointers — zero heap allocations for positional-only calls:

```c
// METH_FASTCALL: vectorcall for extension methods
static PyObject *
fast_add(PyObject *self, PyObject *const *args, Py_ssize_t nargs) {
    if (nargs != 2) { /* error */ }
    long a = PyLong_AsLong(args[0]);  // Borrowed references
    long b = PyLong_AsLong(args[1]);  // No tuple unpacking
    return PyLong_FromLong(a + b);
}
```

`PY_VECTORCALL_ARGUMENTS_OFFSET` signals that `args[-1]` is a writable slot, enabling bound methods to prepend `self` without allocation. All builtin functions, methods, type calls, and Python `def` functions use vectorcall as of 3.12+.

---

## 5. Cache-aware design turns 30-second loops into 10ms operations

### Why Python's object model defeats hardware prefetchers

A Python `list` is an array of `PyObject*` pointers, each pointing to a heap-allocated object scattered across memory. Each integer costs ~40 bytes (28 for the object + 8 for the pointer + padding) versus **8 bytes** in a numpy `float64` array — a 5× memory overhead that also destroys spatial locality. CPU prefetchers cannot predict the next address when chasing pointers.

### Struct-of-Arrays dominates Array-of-Structs for columnar access

When computing kinetic energy across 1M particles, the SoA pattern (separate numpy arrays per field) is **50–200× faster** than AoS (list of Python objects):

```python
# AoS: ~0.5s for kinetic energy computation
ke = sum(0.5 * (p.vx**2 + p.vy**2 + p.vz**2) for p in particles)

# SoA: ~0.003s — vectorized, cache-friendly, SIMD-eligible
ke = (0.5 * (vx**2 + vy**2 + vz**2)).sum()
```

NumPy structured arrays sit between these: they pack records contiguously (good for per-record operations, C interop) but field access returns non-contiguous views with stride equal to the record size, making per-field reductions **2–5× slower** than separate arrays.

### Contiguity and memory order determine SIMD eligibility

C-contiguous (row-major) arrays enable optimal vectorization for row-wise operations; F-contiguous (column-major) for column-wise. Accessing along the non-contiguous dimension is typically **~2× slower** for large arrays. Always verify with `arr.flags['C_CONTIGUOUS']` and convert with `np.ascontiguousarray()` before performance-critical passes.

---

## 6. Profiling from Python bytecodes down to perf counters

### The profiling stack, from highest to lowest level

**cProfile** (deterministic, stdlib) hooks every function call/return via `sys.setprofile`. Overhead scales with call count (2–10×), but provides exact `ncalls`, `tottime`, and `cumtime`. Use `pstats` for post-hoc analysis:

```python
import cProfile, pstats, io
with cProfile.Profile() as pr:
    expensive_computation()
stats = pstats.Stats(pr).strip_dirs().sort_stats('cumulative')
stats.print_stats(20)
```

**py-spy** (sampling, out-of-process) reads the target process's memory at ~100 Hz with ~1–2% overhead. Attach to production processes without restart, generate flamegraphs directly:

```bash
py-spy record -o profile.svg --pid 12345
py-spy record -f speedscope -o profile.json --native --gil -- python app.py
```

**Linux perf** (hardware counters). Python 3.12+ writes JIT trampoline stubs and `/tmp/perf-<PID>.map` files so `perf` can resolve Python function names:

```bash
PYTHONPERFSUPPORT=1 perf record -F 9999 -g python my_script.py
perf script | stackcollapse-perf.pl | flamegraph.pl > flame.svg
```

**line_profiler** provides line-by-line timing within decorated functions. The `@profile` decorator (with `LINE_PROFILE=1` env var) reports hits, time, and percentage per line.

**tracemalloc** tracks every Python-level allocation by file and line. Snapshot comparison reveals memory leaks:

```python
import tracemalloc
tracemalloc.start(25)
snap1 = tracemalloc.take_snapshot()
# ... suspected leaking code ...
snap2 = tracemalloc.take_snapshot()
for stat in snap2.compare_to(snap1, 'lineno')[:10]:
    print(stat)  # Shows size delta and allocation count per line
```

### sys.monitoring replaces sys.settrace with ~5% overhead (3.12+)

PEP 669 introduced `sys.monitoring` — a bytecode-patching monitoring API that replaces the per-opcode checks of `sys.settrace`. It reduces profiling overhead from **~2000% to ~5%**. Up to 6 tools can coexist, each registering callbacks for specific events (`PY_START`, `PY_RETURN`, `CALL`, `LINE`, etc.). Returning `sys.monitoring.DISABLE` from a callback permanently disables that event at that bytecode offset:

```python
import sys
TOOL_ID = sys.monitoring.PROFILER_ID
sys.monitoring.use_tool_id(TOOL_ID, "my_profiler")
sys.monitoring.register_callback(
    TOOL_ID, sys.monitoring.events.PY_START, on_py_start)
sys.monitoring.set_events(
    TOOL_ID, sys.monitoring.events.PY_START | sys.monitoring.events.PY_RETURN)
```

---

## 7. Async internals and zero-copy shared memory for IPC

### The event loop is a selector + callback queue

The `SelectorEventLoop` wraps `epoll` (Linux), `kqueue` (macOS), or IOCP (Windows). Each `_run_once()` iteration: calculate timeout from next scheduled callback → block on selector → process I/O events into callbacks → execute all ready callbacks. When a coroutine hits `await`, it yields to the event loop, which registers the underlying FD with the selector. `asyncio.Task` drives coroutines by calling `coroutine.send(None)` in its `__step` method.

**uvloop** replaces the Python-level selector loop with libuv's C-level loop, delivering **2–4× throughput improvement** for I/O-bound workloads. Azure Functions adopted uvloop as default for Python 3.13+ in 2025, reporting 15–40% throughput gains.

```python
import uvloop
uvloop.run(main())  # Drop-in replacement (uvloop >= 0.18)
```

### Shared memory eliminates serialization for IPC

`multiprocessing.shared_memory` (3.8+) provides POSIX shared memory segments accessible from multiple processes without pickling. For large numpy arrays, this is **~170× faster** than queue-based IPC:

```python
import numpy as np
from multiprocessing import shared_memory, Process, Lock

def worker(shm_name, shape, dtype, lock):
    shm = shared_memory.SharedMemory(name=shm_name)
    arr = np.ndarray(shape, dtype=dtype, buffer=shm.buf)  # Zero-copy!
    with lock:
        arr += 1  # In-place modification visible to all processes
    shm.close()

if __name__ == '__main__':
    shape, dtype = (1000, 1000), np.float64
    shm = shared_memory.SharedMemory(create=True,
                                      size=int(np.prod(shape) * 8))
    shared_arr = np.ndarray(shape, dtype=dtype, buffer=shm.buf)
    shared_arr[:] = 0

    lock = Lock()
    procs = [Process(target=worker, args=(shm.name, shape, dtype, lock))
             for _ in range(4)]
    for p in procs: p.start()
    for p in procs: p.join()
    print(shared_arr[0, 0])  # 4.0
    shm.close(); shm.unlink()
```

---

## 8. SIMD vectorization through numpy ufuncs and BLAS

### Ufunc dispatch: type resolution to AVX-512

Every numpy ufunc wraps one or more C-level 1-D loops selected by dtype matching. NumPy compiles each operation into multiple SIMD variants at build time (SSE2, AVX2, AVX-512, NEON) via universal intrinsics (`npyv_*`), then selects the best at import time via CPUID:

```python
import numpy._core._multiarray_umath as _mu
print(_mu.__cpu_baseline__)   # e.g., 'SSE SSE2 SSE3 SSSE3 SSE41'
print(_mu.__cpu_dispatch__)   # e.g., 'SSE42 AVX AVX2 AVX512F AVX512_SKX'
```

For `np.add` on float32: SSE2 processes 4 elements/instruction, AVX2 processes 8, AVX-512 processes 16. Using **float32 instead of float64** doubles SIMD throughput for the same register width.

### Custom ufuncs: Numba @vectorize is the practical answer

`np.frompyfunc()` and `np.vectorize()` are Python-level loops — 1000× slower than native ufuncs. Numba's `@vectorize` compiles to true C-speed ufuncs:

```python
from numba import vectorize, float64

@vectorize([float64(float64, float64)], target='parallel')
def fast_hypot(x, y):
    return (x**2 + y**2)**0.5

# 10M elements: ~2-5ms (parallel) vs ~10s (np.frompyfunc)
```

### BLAS delegation makes @ the fastest operation in Python

`np.dot`, `np.matmul`, and `@` delegate to BLAS (OpenBLAS, MKL, or Accelerate) for matrix operations. A 2000×2000 matmul reaches **>100 GFLOPS** — BLAS uses SIMD, multi-threading, and cache-tiling internally. Control thread count with `threadpoolctl`:

```python
from threadpoolctl import threadpool_limits
with threadpool_limits(limits=1, user_api='blas'):
    C = A @ B  # Single-threaded BLAS
```

### Array-oriented patterns that eliminate Python loops

The pairwise distance computation illustrates the spectrum. A nested Python loop over 1000×1000 points takes **~30s**. Broadcasting (`X[:,None,:] - Y[None,:,:]`) takes **~50ms** (600×). The algebraic trick `||x-y||² = ||x||² + ||y||² - 2x·y` using `@` (BLAS) takes **~10ms** (3000×):

```python
def pairwise_distance_blas(X, Y):
    x2 = np.einsum('ij,ij->i', X, X)[:, np.newaxis]
    y2 = np.einsum('ij,ij->i', Y, Y)[np.newaxis, :]
    return np.sqrt(np.abs(x2 + y2 - 2.0 * X @ Y.T))
```

**`np.apply_along_axis` is NOT vectorized** — it's a Python loop over slices. Avoid it. Fall back to Numba/Cython for algorithms with loop-carried dependencies (e.g., exponential smoothing) that cannot be expressed as ufunc compositions.

---

## 9. Zero-copy patterns from memoryview to Apache Arrow

### memoryview slicing is ~250× faster than bytes slicing

Bytes slicing copies data; memoryview slicing returns a view with pointer arithmetic:

```python
data = bytearray(20 * 1024 * 1024)  # 20 MB
view = memoryview(data)

# bytes slice: ~500-5000 µs per 1MB slice (copies)
# memoryview:  ~0.2-1 µs per 1MB slice (pointer + offset)
```

`memoryview.cast()` reinterprets memory layout without copying — essential for parsing binary protocols and socket I/O with `recv_into()`.

### mmap for files larger than RAM

`mmap.mmap()` maps file pages directly into the process address space. Only accessed pages are loaded into RAM — the OS transparently pages data in and out. Combined with numpy:

```python
arr = np.memmap('/tmp/large.dat', dtype=np.float32,
                mode='r', shape=(10000, 10000))
# arr[5000, 5000] loads a single 4KB page
# Full computation works transparently; RSS stays low
```

mmap excels for random access to large files and multi-process sharing. For sequential full scans of small files, `read()` is equivalent or simpler.

### NumPy 2.0 copy semantics and pandas copy-on-write

NumPy 2.0 changed `copy=False` to mean "never copy or raise `ValueError`" (previously it meant "copy only if necessary," now expressed as `copy=None`). Pandas 3.0+ enables **copy-on-write** by default: slicing returns a lazy copy that shares memory until mutation, at which point a physical copy is triggered. This eliminates the longstanding view-vs-copy ambiguity.

### Apache Arrow: the zero-copy interchange lingua franca

Arrow enables zero-copy data exchange between pandas, Polars, DuckDB, and ML frameworks via the Arrow C Data Interface (PyCapsule protocol). Zero-copy works for contiguous numeric data without nulls:

```python
import pyarrow as pa
import numpy as np

arr = np.arange(1_000_000, dtype=np.float64)
arrow_arr = pa.array(arr)                    # Zero-copy
np_back = arrow_arr.to_numpy(zero_copy_only=True)  # Zero-copy back

# IPC with memory mapping — true zero-copy reads, 0 bytes allocated
with pa.memory_map('data.arrow', 'rb') as source:
    table = pa.ipc.open_file(source).read_all()
```

Polars' `read_ipc("data.arrow", memory_map=True)` achieves the same. The Arrow IPC format is the most efficient way to move large datasets between Python processes and libraries without serialization overhead.

---

## Conclusion: a decision framework for Python performance work

The optimizations in this reference span a cost-benefit spectrum. **Measure first** — py-spy and `sys.monitoring` have near-zero overhead and identify where time is actually spent. For memory-bound workloads, `__slots__`, numpy arrays, and SoA layout often suffice. For compute-bound loops, **Numba `@njit` delivers 50–200× speedups** with minimal code changes — it should be the default reach before writing C extensions. Cython's typed memoryviews with `nogil` provide comparable performance with finer control. The vectorcall protocol and `METH_FASTCALL` matter for extension authors writing frequently-called functions. For data interchange, Arrow's zero-copy semantics eliminate the serialization tax that dominates many pipeline workloads.

The free-threaded build (3.14+, supported status) represents the most significant architectural shift since Python 3.0. Its <10% single-threaded overhead makes it viable for production, but the ecosystem — particularly C extensions with global state — needs time to catch up. Monitor py-free-threading.github.io for compatibility status. The CPython JIT remains experimental and delivers modest gains on select workloads; invest in Numba or Cython for immediate, large-magnitude speedups. The compounding effect of these techniques — cache-aware layout, BLAS delegation, GIL release, zero-copy interchange — is multiplicative, not additive.