# PyBrain 6-Pillar POC — Conceptual Tree v2

## Refinement: Cython Compilation, Bytecode/AST Enforcement, Cluster Negotiation

This document supersedes v1. It preserves the full categorical structure, Gospel commandments, ologs, file trees, and Observation Product from v1, and integrates three new computational layers that optimize the primordial brain at the machine level.

The three new layers are not engineering additions bolted onto the categorical program. They ARE categorical constructions.

**Cython** is a functor `Compile: C_Python → C_Native` that maps typed morphisms to C-compiled extensions while preserving the olog structure. **AST/Bytecode** is a functor `Verify: C_Syntax → C_Gospel` that enforces categorical invariants at import time, not just test time. **Cluster Negotiation** is a categorical limit construction that dynamically selects the optimal response from a diagram of available tiers.

---

## Part 0: The Category C_Brain (unchanged from v1)

```
Objects:     {Communication, Location, Sight, Life, Resource, Travel}
Morphisms:   Functors (gateway imports between pillars)
Identity:    Each pillar's __init__.py
Composition: Transitive — if F: A→B and G: B→C then G∘F: A→C
```

### The Dependency DAG

```
         Communication (1)
          ↑    ↑    ↑
     ┌────┘    |    └────┐
     |    Sight (3)      |
     |    ↑    ↑         |
     | ┌──┘    └──┐      |
     | Location(2) Life(4)
     | ↑    ↑      ↑
     | └──┐ └──┐ ┌─┘
     |    Travel (6)
     |       ↑
     └───────┤
        Resource (5)
```

### Construction Order (Revised for Compilation + Enforcement)

```
Step 0:   Logging refactor → lib/logging/ (Observation Product foundation)
Step 0.1: AST/Bytecode enforcement hook (import-time Gospel validation)
Step 0.2: Cython build infrastructure (pyproject.toml, setup extensions)
Step 1:   Config + ruff + olog specs (morphisms.yml, error_codes.yml)
Step 2a:  Resource.Foundation (Common + Exception)
Step 2b:  Resource.Interface (Lib — demand-driven only)
Step 2c:  Cython compilation of Resource hot paths (bitmask, identity)
Step 3:   Travel (depends on Resource only)
Step 3c:  Cython compilation of Travel hot paths (monad bind, composition)
Step 4:   Life (depends on Resource + Travel)
Step 4c:  Cython compilation of Life hot paths (Observation Product writers)
Step 5:   Location (depends on Resource + Travel + Life)
Step 6:   Sight (depends on Resource + Travel + Life + Location)
Step 7:   Communication (depends on ALL)
Step 8:   Gospel Tests (architecture + unit + integration)
Step 9:   Three-tier vector integration (flash/reference/memory)
Step 9c:  Cython compilation of embedding distance computations
Step 10:  Cluster negotiation layer (dynamic tier dispatch)
```

---

## Part 1: Gospel Commandments (Extended)

Gospels XIV through XX are unchanged from v1. Three new Gospels are added.

### Gospel XXI: Compiled Morphism (Cython Rule)

Every morphism that meets ALL of the following criteria SHALL have a `.pyx` Cython implementation alongside its `.py` source. The `.py` file remains the canonical olog declaration. The `.pyx` file is the compiled functor image.

Criteria for Cython compilation:
1. The function is concrete (not abstract).
2. The function is called more than once per session in expected operation.
3. The function's input and output types are statically resolvable.
4. The function contains no dynamic dispatch that would defeat C-level typing.

The `.pyx` file MUST preserve the exact same olog semantics as the `.py` file. The Cython compilation is a faithful functor: it changes performance, not behavior.

```
app/resource/common/_base.pyx       ← compiled from base.py hot paths
app/travel/mechanism/monad/_result.pyx  ← compiled Result.bind / fmap
app/life/output/_duckdb.pyx         ← compiled triple-writer hot path
```

### Gospel XXII: Verified Import (AST/Bytecode Rule)

Every module import within `app/` SHALL pass through a meta-path finder that performs AST-level Gospel validation at import time. Violations are raised as `GospelViolationError` immediately, before the module executes. This makes Gospel compliance a runtime invariant, not merely a test assertion.

The import hook verifies:
1. Gospel XIV — function body statement count ≤ 10
2. Gospel XV — parameter count ≤ 1 (excl self/cls)
3. Gospel XVI — first statement is Observer producer witness call (Cython-powered via RingBuffer)
4. Gospel XVIII — import sources comply with depth factoring laws

### Gospel XXIII: Cluster Quorum (Concurrency Rule)

Every concurrent write group (the Observation Product's three writers, any multi-threaded pillar operation) SHALL operate as a defined cluster with explicit membership, sequence consensus, and fault reporting. No ad-hoc threading. Every thread pool is a declared object in the olog with a known cardinality.

---

## Part 2: The UNIX Ontology (unchanged from v1)

```
SYSTEM=/sys,/proc  NETWORK=/dev  DATABASE=/var/lib  SECURITY=/etc
UI=/usr  CORE=/lib,/bin  PERFORMANCE=/var/log  INTEGRATION=/opt,/mnt
```

8 × 8 = 64-bit bitmask. Per-file category assignment by pillar. See v1 for complete mapping.

---

## Part 3: The Cython Compilation Layer

### 3.1 Categorical Foundation

Cython compilation is a functor `Compile: C_Python → C_Native`. For every Python module M in C_Brain that meets Gospel XXI criteria, `Compile(M)` produces a shared object (`.so`) that is object-code equivalent. The functor preserves composition: if `f: A → B` and `g: B → C` are both compiled, then `Compile(g ∘ f) = Compile(g) ∘ Compile(f)`.

The key insight: the Gospel constraints make Cython compilation trivial. Gospel XV (1-in/1-out) means every function has exactly one typed parameter and one typed return. Gospel XIV (10 lines) means function bodies are small enough for Cython to inline entirely. Frozen dataclass products (Gospel XV's multi-input pattern) map directly to C structs.

### 3.2 Compilation Targets (Ordered by Hotness)

#### Tier 1: Bitmask Engine (Step 2c)

The 64-bit category bitmask is the hottest path in the brain. Every single CALLED emission touches it. The `LogCategoryManager` and `LogCategory` IntFlag operations are pure bitwise arithmetic on `uint64_t`. This is the first Cython target.

```
File: app/resource/lib/logging/helpers/types/category/_ops.pyx

# cython: language_level=3
# cython: boundscheck=False
# cython: wraparound=False

from libc.stdint cimport uint64_t

cdef inline bint mask_matches(uint64_t event_mask, uint64_t filter_mask) nogil:
    """Check if event_mask has ANY bit in common with filter_mask."""
    return (event_mask & filter_mask) != 0

cdef inline uint64_t combine_masks(uint64_t parent, uint64_t child) nogil:
    """Combine a parent group mask with a child sub-category mask."""
    return parent | child

cpdef bint check_category(uint64_t event_mask, uint64_t filter_mask):
    """Python-callable bitmask check. Gospel XVI exempt (leaf operation)."""
    return mask_matches(event_mask, filter_mask)

cpdef uint64_t full_mask(uint64_t group_bit, uint64_t sub_bit):
    """Compute full 64-bit mask from group + sub-category."""
    return combine_masks(group_bit, sub_bit)
```

The corresponding `.pxd` declaration file:

```
File: app/resource/lib/logging/helpers/types/category/_ops.pxd

from libc.stdint cimport uint64_t

cdef inline bint mask_matches(uint64_t event_mask, uint64_t filter_mask) nogil
cdef inline uint64_t combine_masks(uint64_t parent, uint64_t child) nogil
cpdef bint check_category(uint64_t event_mask, uint64_t filter_mask)
cpdef uint64_t full_mask(uint64_t group_bit, uint64_t sub_bit)
```

#### Tier 2: Monad Operations (Step 3c)

The `Result` monad's `bind` and `fmap` are called on every morphism that returns a Result. The monadic operations are the composition engine of Travel. With typed `Success[T]` and `Failure[E]`, Cython can eliminate Python dispatch overhead.

```
File: app/travel/mechanism/monad/_result.pyx

# cython: language_level=3

cdef class CResult:
    cdef bint _is_success
    cdef object _value

    def __cinit__(self, bint is_success, object value):
        self._is_success = is_success
        self._value = value

    cpdef CResult bind(self, object f):
        """Monadic bind. If Success, apply f. If Failure, propagate."""
        if self._is_success:
            return f(self._value)
        return self

    cpdef CResult fmap(self, object f):
        """Functor map. Apply f to value if Success."""
        if self._is_success:
            return CResult(True, f(self._value))
        return self

    @staticmethod
    cpdef CResult pure(object value):
        """Monadic return / unit."""
        return CResult(True, value)

    @staticmethod
    cpdef CResult fail(object error):
        """Construct a Failure."""
        return CResult(False, error)

    @property
    def is_success(self) -> bool:
        return self._is_success

    @property
    def value(self) -> object:
        return self._value
```

#### Tier 3: Observation Product Writers (Step 4c)

The three DuckDB writer threads serialize data on every CALLED emission. The serialization of the Avro-equivalent record, the assembly of the graph edge tuple, and the embedding distance computation during flash reads are all hot paths.

```
File: app/life/output/_serialize.pyx

# cython: language_level=3
from libc.stdint cimport uint64_t, int64_t
from cpython.bytes cimport PyBytes_FromStringAndSize

cdef struct TraceRecord:
    int64_t sequence_id
    uint64_t category_mask
    # remaining fields handled as Python objects for flexibility

cpdef tuple pack_avro_record(
    object session_id,
    int64_t sequence_id,
    str source_module,
    str pillar,
    uint64_t category_mask,
    str function_name,
    str class_name,
    str argument_summary,
):
    """Pack a CALLED emission into an Avro-equivalent insert tuple.

    Returns a tuple ready for DuckDB parameterized INSERT.
    """
    return (
        session_id, sequence_id, source_module, pillar,
        category_mask, function_name, class_name, argument_summary,
    )

cpdef tuple pack_graph_edge(
    object session_id,
    int64_t from_seq,
    int64_t to_seq,
    str edge_type,
):
    """Pack a graph edge into an insert tuple."""
    return (session_id, from_seq, to_seq, edge_type)
```

#### Tier 4: Vector Distance (Step 9c)

The cosine similarity computation during flash-tier reads. While DuckDB handles this in SQL for indexed queries, there are code paths (pre-filter, threshold checks) where Python-side distance computation is needed.

```
File: app/life/output/_distance.pyx

# cython: language_level=3
# cython: boundscheck=False
# cython: wraparound=False

from libc.math cimport sqrtf

cpdef float cosine_similarity(float[:] a, float[:] b):
    """Compute cosine similarity between two FLOAT[384] vectors."""
    cdef int i
    cdef int n = a.shape[0]
    cdef float dot = 0.0, norm_a = 0.0, norm_b = 0.0

    for i in range(n):
        dot += a[i] * b[i]
        norm_a += a[i] * a[i]
        norm_b += b[i] * b[i]

    cdef float denom = sqrtf(norm_a) * sqrtf(norm_b)
    if denom == 0.0:
        return 0.0
    return dot / denom
```

### 3.3 Build Infrastructure

```toml
# pyproject.toml additions

[build-system]
requires = ["setuptools>=68", "cython>=3.0"]
build-backend = "setuptools.build_meta"

[tool.setuptools.ext-modules]
# Declarative Cython extension list
# Each .pyx file that passes Gospel XXI criteria
extensions = [
    {name = "app.resource.lib.logging.helpers.types.category._ops", sources = ["app/resource/lib/logging/helpers/types/category/_ops.pyx"]},
    {name = "app.travel.mechanism.monad._result", sources = ["app/travel/mechanism/monad/_result.pyx"]},
    {name = "app.life.output._serialize", sources = ["app/life/output/_serialize.pyx"]},
    {name = "app.life.output._distance", sources = ["app/life/output/_distance.pyx"]},
]
```

### 3.4 Fallback Rule

Every `.pyx` file has a corresponding pure-Python fallback. The `.py` file IS the olog. The `.pyx` file IS the compiled image. Imports use a try/except pattern:

```python
# In app/resource/lib/logging/helpers/types/category/__init__.py
try:
    from app.pillars.resource.lib.logging.helpers.types.category._ops import check_category, full_mask
except ImportError:
    from app.pillars.resource.lib.logging.helpers.types.category.manager import check_category, full_mask
```

If Cython is not available (development machine, CI without build tools), the pure-Python path executes. The olog is never dependent on compilation. Compilation is an optimization functor, not a structural requirement.

### 3.5 What NOT to Cythonize

Do not compile: abstract methods (they are declarations, never called directly), `__init__.py` gateways (they are identity morphisms with no computation), any module whose functions have dynamic dispatch that cannot be statically typed, any module under active development (Cython compilation adds build latency — compile only when the olog for that module is stable).

---

## Part 4: The AST/Bytecode Enforcement Layer

### 4.1 Categorical Foundation

Python's AST is a syntax category: objects are AST node types (`FunctionDef`, `ClassDef`, `Import`, etc.), morphisms are parent-child relationships in the tree. An AST visitor is a functor from this syntax category to some target category. The Gospel tests in v1 were functors from C_Syntax to C_Bool (pass/fail). The enforcement layer upgrades this to a functor from C_Syntax to C_Gospel that fires at import time.

The bytecode layer goes one level deeper. After the AST is compiled to bytecode, the `dis` module can inspect the actual instruction sequence. Bytecode verification confirms that the Observer witness call is not merely present in the AST but is the first executed instruction — accounting for decorator wrappers, conditional compilation, and other transformations that might reorder instructions between AST and execution.

### 4.2 The Import Hook

```python
# File: app/resource/lib/ast/__init__.py (added to lib/ because AST enforcement needs it)
# Also: app/_boot.py — the meta-path finder registration

import ast
import sys
import importlib.abc
import importlib.machinery

class GospelViolationError(ImportError):
    """Raised when a module violates Gospel constraints at import time."""
    pass

class GospelMetaFinder(importlib.abc.MetaPathFinder):
    """Meta-path finder that validates Gospel compliance on every app/ import."""

    def find_module(self, fullname, path=None):
        if fullname.startswith("app."):
            return GospelLoader()
        return None

class GospelLoader(importlib.abc.Loader):
    """Loader that AST-validates before executing the module."""

    def exec_module(self, module):
        source_path = module.__spec__.origin
        if source_path and source_path.endswith(".py"):
            with open(source_path, "r") as f:
                source = f.read()
            tree = ast.parse(source, filename=source_path)
            violations = self._validate(tree, source_path)
            if violations:
                msg = f"Gospel violations in {source_path}:\n" + "\n".join(violations)
                raise GospelViolationError(msg)

    def _validate(self, tree, path):
        violations = []
        for node in ast.walk(tree):
            if isinstance(node, ast.FunctionDef):
                violations.extend(self._check_function(node, path))
        return violations

    def _check_function(self, node, path):
        issues = []
        name = node.name

        # Skip dunders, properties, abstractmethods
        if name.startswith("__") and name.endswith("__"):
            return issues
        if any(
            isinstance(d, ast.Name) and d.id in ("abstractmethod", "property")
            for d in node.decorator_list
        ):
            return issues
        if any(
            isinstance(d, ast.Attribute) and d.attr in ("abstractmethod", "property")
            for d in node.decorator_list
        ):
            return issues

        # Gospel XIV: body ≤ 10 statements
        body_stmts = [s for s in node.body if not isinstance(s, (ast.Pass,))]
        # Exclude the docstring
        if body_stmts and isinstance(body_stmts[0], ast.Expr) and isinstance(body_stmts[0].value, ast.Constant):
            body_stmts = body_stmts[1:]
        if len(body_stmts) > 10:
            issues.append(
                f"  XIV: {path}:{node.lineno} {name}() has {len(body_stmts)} stmts (max 10)"
            )

        # Gospel XV: params ≤ 1 (excl self/cls)
        args = node.args
        param_count = len(args.args)
        if args.args and args.args[0].arg in ("self", "cls"):
            param_count -= 1
        if param_count > 1:
            issues.append(
                f"  XV: {path}:{node.lineno} {name}() has {param_count} params (max 1)"
            )

        # Gospel XVI: first stmt is Observer producer witness call
        # Only for non-abstract, non-property, non-dunder concrete functions
        if body_stmts:
            first = body_stmts[0]
            if not self._is_observer_witness(first):
                # Check if it's a raise NotImplementedError (abstract body)
                if not (isinstance(first, ast.Raise)):
                    issues.append(
                        f"  XVI: {path}:{node.lineno} {name}() missing Observer witness as first stmt"
                    )

        return issues

    def _is_observer_witness(self, stmt):
        """Check if statement is Observer producer witness call."""
        if not isinstance(stmt, ast.Expr):
            return False
        call = stmt.value
        if not isinstance(call, ast.Call):
            return False
        func = call.func
        if isinstance(func, ast.Attribute):
            if func.attr == "witness":
                return True
        return False

# Registration — called from app/__init__.py or conftest.py
def install_gospel_enforcement():
    sys.meta_path.insert(0, GospelMetaFinder())
```

### 4.3 Bytecode Verification (Deeper Than AST)

AST validation catches structural violations but cannot detect runtime reordering (decorators that wrap functions, metaclasses that modify methods). Bytecode verification uses `dis` to confirm that the compiled code object's first meaningful instruction sequence corresponds to the Observer witness call.

```python
# File: app/tests/architecture/test_gospel_xvi_bytecode.py

import dis
import types

def verify_observer_witness_bytecode(func: types.FunctionType) -> bool:
    """Verify that the FIRST instruction sequence in the bytecode
    is an Observer producer witness call chain.

    This catches cases where decorators or metaclasses reorder the AST.
    """
    instructions = list(dis.get_instructions(func))

    # Skip past any initial RESUME instruction (Python 3.11+)
    start = 0
    for i, instr in enumerate(instructions):
        if instr.opname == "RESUME":
            start = i + 1
            break

    # Look for the pattern: LOAD observer, LOAD_ATTR witness
    # within the first 10 instructions after RESUME
    window = instructions[start:start + 10]
    for i, instr in enumerate(window):
        if instr.opname in ("LOAD_GLOBAL", "LOAD_NAME"):
            # Next should be LOAD_ATTR witness
            if i + 1 < len(window) and window[i + 1].opname == "LOAD_ATTR":
                if window[i + 1].argval == "witness":
                    return True
    return False
```

### 4.4 AST as Olog Introspection

Beyond enforcement, the AST layer provides the mechanism for the brain to introspect its own structure. An AST visitor that walks all `.py` files under `app/` can extract the complete morphism graph: every function definition is an arrow, every import is a dependency edge, every class definition is an object. This extracted graph can be compared against the declared olog in `morphisms.yml` to verify olog fidelity automatically.

```python
# File: app/tests/architecture/test_gospel_xix_olog_only.py

import ast
import yaml

def extract_morphism_graph(app_root: str) -> dict:
    """Walk all .py files, extract functions as arrows, classes as objects."""
    graph = {"objects": set(), "arrows": []}
    for py_file in Path(app_root).rglob("*.py"):
        tree = ast.parse(py_file.read_text(), filename=str(py_file))
        for node in ast.walk(tree):
            if isinstance(node, ast.ClassDef):
                graph["objects"].add(node.name)
            if isinstance(node, ast.FunctionDef):
                graph["arrows"].append({
                    "name": node.name,
                    "file": str(py_file),
                    "line": node.lineno,
                })
    return graph

def test_olog_fidelity():
    """Every class and method in code traces to a declared olog entry."""
    declared = yaml.safe_load(open("app/config/ologs.yml"))
    actual = extract_morphism_graph("app/")
    for obj in actual["objects"]:
        assert obj in declared["all_objects"], f"Orphan object: {obj}"
    for arrow in actual["arrows"]:
        # Skip dunders and private helpers
        if arrow["name"].startswith("_"):
            continue
        assert arrow["name"] in declared["all_morphisms"], (
            f"Orphan morphism: {arrow['name']} in {arrow['file']}:{arrow['line']}"
        )
```

### 4.5 The ologs.yml Config (New)

The olog declarations from all six pillars are collected into a single machine-readable file that the AST enforcement layer validates against.

```yaml
# File: app/config/ologs.yml

all_objects:
  # Resource
  - AbstractThing
  - Identity
  - LogLevel
  - ABCEnumMeta
  - ThingException
  # Travel
  - Thing
  - AbstractType
  - AbstractAction
  - AbstractCollision
  - AbstractLifecycle
  - AbstractMechanism
  - AbstractComposition
  - AbstractConstraint
  - AbstractLogic
  - AbstractMonad
  - AbstractOperation
  - AbstractLanguage
  - AbstractGrammar
  - AbstractLexicon
  - AbstractSemantics
  - AbstractSerde
  - AbstractFormat
  - AbstractTranslator
  - AbstractFactory
  # Life
  - AbstractObserver
  - AbstractInstrument
  - AbstractEvent
  - AbstractOutput
  - AbstractSchema
  # Location
  - AbstractCoordinate
  - AbstractAddress
  - AbstractProjection
  - AbstractCRS
  - AbstractPlatform
  - LocalPlatform
  - SystemPlatform
  - RealmPlatform
  - SandboxPlatform
  - AbstractExternal
  - WorldExternal
  - DomainExternal
  # Sight
  - AbstractTransform
  - AbstractGenerate
  - AbstractView
  # Communication
  - AbstractCommunication
  - AbstractParticipant
  - AbstractBroker
  - AbstractProducer
  - AbstractConsumer
  - AbstractCommunicationLanguage
  - AbstractSyntax
  - AbstractDialect
  - AbstractCommunicationFormat
  # Observation Product
  - ObservationProductHandler
  - DuckDBOutput
  - TraceCluster
  # Cluster
  - AbstractCluster
  - WriterCluster
  - TierNegotiator

all_morphisms:
  # Sacred methods (AbstractThing)
  - identify
  - describe
  - validate
  - snapshot
  # Resource
  # Travel
  - apply
  - pipe
  - compose
  - satisfies
  - evaluate
  - pure
  - bind
  - fmap
  - encode
  - decode
  - parse
  - lookup
  - register
  - interpret
  - resolve
  - serialize
  - deserialize
  - to_wire
  - from_wire
  - translate
  - create
  # Life
  - observe
  - record
  - measure
  - emit
  - write
  - flush
  # Location
  - distance
  - project
  - inverse
  - transform
  - boot
  - shutdown
  - status
  - connect
  - disconnect
  # Sight
  - render
  - refresh
  - produce
  # Communication
  - send
  - receive
  - mediate
  - route
  - filter
  - publish
  - process
  - watch
  - tokenize
  - structure
  - adapt
  # Observation Product
  - check_category
  - full_mask
  - cosine_similarity
  - pack_avro_record
  - pack_graph_edge
  # Cluster
  - negotiate
  - elect_leader
  - report_health
  - dispatch
```

---

## Part 5: The Cluster Negotiation Layer

### 5.1 Categorical Foundation

A cluster is a small category whose objects are nodes (threads/processes/tiers) and whose morphisms are communication channels between them. A cluster with N nodes has a distinguished morphism `elect: Cluster → Node` that selects a leader — this is a limit construction (the terminal cone over the diagram of available nodes).

Dynamic negotiation is a parametric limit: given a query Q and the current state of all tiers (load, confidence, latency), the negotiator computes the limit of the diagram:

```
         Flash ──capability──→ Response
           ↑                     ↑
      Reference ──capability──→ Response
           ↑                     ↑
        Memory ──capability──→ Response
```

The limit is the tier that provides the best response given the constraints. This is not a fixed hierarchy (always try flash first) — it is a dynamic computation that accounts for cache state, index freshness, and confidence thresholds.

### 5.2 The Three Cluster Types

#### 5.2.1 Writer Cluster (Observation Product)

The three writer threads (Avro, Vector, Graph) form a cluster of cardinality 3. They require sequence consensus: all three must agree on the `sequence_id` for each CALLED emission before writing.

```
┌─────────────────────────────────────────────────────────────────┐
│                     Writer Cluster (N=3)                        │
│                                                                 │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐                   │
│   │  Avro   │    │ Vector  │    │  Graph  │                   │
│   │ Writer  │    │ Writer  │    │ Writer  │                   │
│   └────┬────┘    └────┬────┘    └────┬────┘                   │
│        │              │              │                         │
│        └──────────┬───┘──────────────┘                         │
│                   ▼                                             │
│         ┌─────────────────┐                                    │
│         │ SequenceCounter │ ← AtomicCounter (single authority) │
│         │  (the leader)   │                                    │
│         └─────────────────┘                                    │
│                                                                 │
│   Consensus: Leader assigns seq_id BEFORE fan-out to writers.  │
│   Fault: If any writer fails, the others complete. The failed  │
│          trace is logged to a dead-letter table for replay.    │
│   Health: Each writer reports latency to the cluster monitor.  │
└─────────────────────────────────────────────────────────────────┘
```

The sequence counter is the leader. It is not elected — it is structurally determined (there is exactly one counter per session). The leader assigns `sequence_id` atomically, then fans out to the three writers. The writers do not coordinate with each other — they coordinate only with the leader. This is not Raft; it is simpler. It is a star topology with a fixed center.

Fault tolerance: if the Vector writer fails (embedding model unavailable), the Avro and Graph writers still complete. The missing vector trace is logged to a `trace_dead_letters` table with enough information to replay the embedding when the model recovers. This guarantees that the Avro and Graph representations are never blocked by a vector failure.

```sql
CREATE TABLE trace_dead_letters (
    session_id       UUID NOT NULL,
    sequence_id      BIGINT NOT NULL,
    failed_trace     VARCHAR NOT NULL,  -- 'avro' | 'vector' | 'graph'
    error_message    VARCHAR,
    retry_count      INTEGER DEFAULT 0,
    created_at       TIMESTAMP DEFAULT current_timestamp
);
```

#### 5.2.2 Tier Cluster (Vector Search)

The three vector tiers (Flash, Reference, Memory) form a cluster that negotiates which tier handles each query. Unlike the Writer Cluster (fixed star topology), the Tier Cluster uses dynamic dispatch based on confidence and latency.

```
┌─────────────────────────────────────────────────────────────────┐
│                   Tier Cluster (N=3)                            │
│                                                                 │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐                  │
│   │  Flash   │   │Reference │   │  Memory  │                  │
│   │  384-dim │   │  768-dim │   │ 1024-dim │                  │
│   │  <1ms    │   │  ~5ms    │   │  ~20ms   │                  │
│   └────┬─────┘   └────┬─────┘   └────┬─────┘                  │
│        │              │              │                         │
│        └──────────┬───┘──────────────┘                         │
│                   ▼                                             │
│         ┌─────────────────┐                                    │
│         │  TierNegotiator │ ← dynamic dispatch                 │
│         └─────────────────┘                                    │
│                                                                 │
│   Protocol:                                                     │
│   1. Query arrives at negotiator                               │
│   2. Flash responds with top-k + confidence score              │
│   3. If confidence ≥ threshold → return flash results          │
│   4. If confidence < threshold → escalate to Reference         │
│   5. If Reference confidence < threshold → escalate to Memory  │
│   6. If Memory misses → fall through to graph traversal        │
│                                                                 │
│   Confidence = max(cosine_similarity) of top-k results         │
│   Threshold is configurable per-category (rootfs group)        │
└─────────────────────────────────────────────────────────────────┘
```

The negotiation protocol is a cascade, analogous to the CPU cache miss cascade. But unlike a CPU cache (which has fixed miss policy), the tier negotiator can make dynamic decisions based on the category of the query. A SECURITY-category query might require Memory-tier resolution (high dimensional fidelity for trust boundary decisions), while a PERFORMANCE-category query can be served from Flash (approximate is fine for telemetry context).

```python
# File: app/life/output/negotiator.py

@dataclass(frozen=True)
class TierResult:
    """Result from a single tier query."""
    tier: str
    top_k: list
    confidence: float
    latency_ms: float

@dataclass(frozen=True)
class NegotiationPolicy:
    """Per-category confidence thresholds."""
    flash_threshold: float = 0.75
    reference_threshold: float = 0.85
    memory_threshold: float = 0.90

class TierNegotiator:
    """Dispatches vector queries across flash/reference/memory tiers."""

    def dispatch(self, request: DispatchRequest) -> TierResult:
        witness(type(self).__name__, "dispatch", category="DATABASE")

        policy = self._policy_for_category(request.category_mask)

        # Flash first (always)
        flash_result = self._query_flash(request)
        if flash_result.confidence >= policy.flash_threshold:
            return flash_result

        # Escalate to Reference
        ref_result = self._query_reference(request)
        if ref_result.confidence >= policy.reference_threshold:
            return ref_result

        # Escalate to Memory
        mem_result = self._query_memory(request)
        return mem_result  # Best we have — return regardless of confidence
```

#### 5.2.3 Pillar Cluster (Future — Session Parallelism)

When multiple sessions run concurrently (multiple users, multiple agent threads), each session's Observation Product operates independently. But cross-session queries (such as "has ANY session seen content similar to this?") require inter-session coordination. This is the Pillar Cluster: a cluster of session-level brain instances that can federate their graph and vector stores.

This is not implemented at Steps 0–9. It is declared here as a structural placeholder for Phase 2 (Realm) and Phase 3 (Sandbox Workflows) from the project phases. The categorical structure is: a Pillar Cluster is a category of categories, where each object is a session-local C_Brain instance and morphisms are inter-session functors (query delegation, result aggregation).

### 5.3 Cluster as Olog Object

Clusters are first-class olog objects. They live in the Life pillar (observation capacity) because clusters are the mechanism by which the brain coordinates its internal observation.

```
New objects in C_Life:
  Cluster              "a coordinated group of workers"
    WriterCluster      "the Observation Product's three writers"
    TierCluster        "the vector search tier negotiation group"
    PillarCluster      "cross-session federation" (Phase 2+)

New morphisms in C_Life:
  Cluster → Life                   "coordinates within"
  WriterCluster → Cluster          "writes as"
  TierCluster → Cluster            "negotiates as"
  PillarCluster → Cluster          "federates as"
  WriterCluster → Output           "writes to" (3 trace tables)
  TierCluster → Instrument         "measures with" (tier selection)
```

### 5.4 DuckDB Threading Model for Clusters

```python
# Cluster-aware DuckDB connection management

class ClusterConnection:
    """Manages DuckDB cursors for a cluster of N worker threads."""

    def __init__(self, db_connection, cluster_size: int):
        witness(type(self).__name__, "__init__", category="DATABASE")
        self._db = db_connection
        self._cursors = {}
        self._cluster_size = cluster_size

    def cursor_for(self, request: CursorRequest) -> duckdb.DuckDBPyConnection:
        """Return a thread-local cursor for the given worker_id."""
        witness(type(self).__name__, "cursor_for", category="DATABASE")
        worker_id = request.worker_id
        if worker_id not in self._cursors:
            self._cursors[worker_id] = self._db.cursor()
        return self._cursors[worker_id]
```

Each worker in a cluster gets its own cursor. The DuckDB documentation confirms this is safe: each thread creates a thread-local connection via `.cursor()` on the shared base connection. The `ClusterConnection` object manages the cursor pool and provides the `cursor_for` morphism that maps a worker identity to its DuckDB handle.

### 5.5 Health Reporting and Dead-Letter Replay

Every cluster member reports health on a heartbeat interval. Health is a simple struct: latency of last operation, error count since last report, queue depth.

```python
@dataclass(frozen=True)
class HealthReport:
    worker_id: str
    latency_ms: float
    error_count: int
    queue_depth: int
    timestamp: float
```

If a worker's error count exceeds a threshold, the cluster leader marks it as degraded. Degraded workers stop receiving new work. A background thread replays dead letters when the worker recovers. This is the fault tolerance arm of the Rule of Threes: the system continues operating with 2-of-3 writers while the third recovers.

---

## Part 6: Pillar Ologs and File Trees (Updated)

All pillar ologs from v1 remain unchanged. The following files are ADDED to the file trees declared in v1.

### Resource (additions)

```
app/resource/lib/logging/helpers/types/category/
    _ops.pyx        # Cython: bitmask operations (Gospel XXI)
    _ops.pxd        # Cython header

app/resource/lib/ast/
    __init__.py     # Gateway: AST enforcement utilities
    hook.py         # GospelMetaFinder + GospelLoader (Gospel XXII)
    visitor.py      # Olog extraction visitor (Gospel XIX verification)
```

### Travel (additions)

```
app/travel/mechanism/monad/
    _result.pyx     # Cython: Result.bind/fmap (Gospel XXI)
    _result.pxd     # Cython header
```

### Life (additions)

```
app/life/output/
    _serialize.pyx  # Cython: trace record packing (Gospel XXI)
    _distance.pyx   # Cython: cosine similarity (Gospel XXI)
    _serialize.pxd  # Cython header
    _distance.pxd   # Cython header
    negotiator.py   # TierNegotiator: dynamic tier dispatch
    cluster.py      # WriterCluster + ClusterConnection
    dead_letter.py  # Dead-letter table management and replay
```

### Config (additions)

```
app/config/
    ologs.yml       # Machine-readable olog object + morphism registry
    clusters.yml    # Cluster definitions (cardinality, thresholds, policies)
```

### Tests (additions)

```
app/tests/architecture/
    test_gospel_xvi_bytecode.py    # Bytecode-level Observer witness verification
    test_gospel_xix_olog_only.py   # AST ↔ ologs.yml fidelity check
    test_gospel_xxi_cython.py      # Verify .pyx files preserve .py semantics
    test_gospel_xxii_import.py     # Verify import hook catches violations
    test_gospel_xxiii_cluster.py   # Verify cluster membership + consensus

app/tests/unit/life/
    test_negotiator.py             # Tier dispatch logic
    test_writer_cluster.py         # 3-writer consensus + fault tolerance
    test_dead_letter.py            # Replay mechanics

app/tests/integration/
    test_cython_fallback.py        # Compiled vs pure-Python equivalence
    test_cluster_degradation.py    # 2-of-3 writer survival
    test_tier_cascade.py           # Flash → Reference → Memory escalation
```

---

## Part 7: clusters.yml Configuration

```yaml
# File: app/config/clusters.yml

writer_cluster:
  name: "observation_product_writers"
  cardinality: 3
  members:
    - id: "avro_writer"
      trace_table: "trace_avro"
      rootfs_category: "NETWORK"
      required: true          # Must succeed for session integrity
    - id: "vector_writer"
      trace_table: "trace_vectors"
      rootfs_category: "DATABASE"
      required: false         # Can fail; dead-lettered for replay
    - id: "graph_writer"
      trace_table: "trace_graph_nodes"
      rootfs_category: "PERFORMANCE"
      required: true          # Must succeed for composition chain integrity
  consensus:
    type: "leader_assigned"
    leader: "sequence_counter"
    ordering: "monotonic_increment"
  fault_tolerance:
    min_writers: 2
    dead_letter_table: "trace_dead_letters"
    max_retry: 3
    retry_backoff_ms: [100, 500, 2000]

tier_cluster:
  name: "vector_tier_negotiation"
  cardinality: 3
  members:
    - id: "flash"
      model: "granite-embedding-small-english-r2"
      dimensions: 384
      table: "trace_vectors"
      index: "flash_idx"
      latency_target_ms: 1
    - id: "reference"
      model: "granite-embedding-english-r2"
      dimensions: 768
      table: "trace_vectors_ref"
      index: "reference_idx"
      latency_target_ms: 5
    - id: "memory"
      model: "TBD"
      dimensions: 1024
      table: "trace_vectors_mem"
      index: "memory_idx"
      latency_target_ms: 20
  negotiation:
    type: "cascade_with_confidence"
    thresholds:
      default:
        flash: 0.75
        reference: 0.85
      SECURITY:
        flash: 0.90        # Security queries demand high confidence
        reference: 0.95
      PERFORMANCE:
        flash: 0.60        # Telemetry queries accept lower confidence
        reference: 0.70

pillar_cluster:
  name: "cross_session_federation"
  cardinality: "dynamic"     # Grows with active sessions
  phase: "2_realm"           # Not implemented until Phase 2
  consensus:
    type: "eventual"         # Eventual consistency across sessions
```

---

## Part 8: Verification Checkpoints (Extended)

All v1 checkpoints remain. Additional checkpoints for the new layers:

### Step 0.1 (AST Enforcement)
```bash
uv run python -c "
from app.pillars.resource.lib.ast.hook import install_gospel_enforcement
install_gospel_enforcement()
# Now any 'from app.xxx import yyy' will be AST-validated
import app.resource
print('Gospel enforcement active')
"
```

### Step 0.2 (Cython Build)
```bash
uv run python setup.py build_ext --inplace
uv run python -c "
from app.pillars.resource.lib.logging.helpers.types.category._ops import check_category
print(check_category(0x01, 0x01))  # True
print(check_category(0x01, 0x02))  # False
print('Cython bitmask OK')
"
```

### Step 2c (Resource Cython)
```bash
uv run python -c "
from app.pillars.resource.lib.logging.helpers.types.category._ops import full_mask
mask = full_mask(0x01, 0x0100)
assert mask == 0x0101
print('Cython full_mask OK')
"
```

### Step 3c (Travel Cython)
```bash
uv run python -c "
from app.travel.mechanism.monad._result import CResult
r = CResult.pure(42)
r2 = r.bind(lambda x: CResult.pure(x + 1))
assert r2.value == 43
print('Cython Result.bind OK')
"
```

### Step 4c (Life Cython)
```bash
uv run python -c "
from app.pillars.life.output._distance import cosine_similarity
import array
a = array.array('f', [1.0, 0.0, 0.0])
b = array.array('f', [1.0, 0.0, 0.0])
assert abs(cosine_similarity(a, b) - 1.0) < 1e-6
print('Cython cosine OK')
"
```

### Step 10 (Cluster Negotiation)
```bash
uv run pytest app/tests/unit/life/test_writer_cluster.py -v
uv run pytest app/tests/unit/life/test_negotiator.py -v
uv run pytest app/tests/integration/test_cluster_degradation.py -v
```

---

## Part 9: The Four Invariants (Extended from Three)

**Invariant 1 — Olog Fidelity**: Every file, class, and method traces to a declared olog object or morphism in `ologs.yml`. Verified by AST extraction at test time and import-time hooks in production.

**Invariant 2 — Gospel Compliance**: Every concrete function has Observer witness call (verified at AST and bytecode level), ≤ 10 statements, ≤ 1 parameter, pillar-scoped exception codes, and lawful imports. Enforced at import time by `GospelMetaFinder`.

**Invariant 3 — Observation Product**: Every CALLED emission produces a triple (avro, vector, graph) with shared `(session_id, sequence_id)`, written to DuckDB via the Writer Cluster with leader-assigned consensus and dead-letter fault tolerance.

**Invariant 4 — Compilation Fidelity**: Every `.pyx` file produces identical outputs to its `.py` counterpart for all inputs. Verified by `test_cython_fallback.py` which runs the same test suite against both compiled and pure-Python implementations.

---

## Part 10: What NOT to Create (Extended)

All prohibitions from v1 remain. Additional prohibitions:

11. Do not Cythonize abstract methods, gateways, or any code that does not meet ALL four Gospel XXI criteria.
12. Do not create `.pyx` files without corresponding `.pxd` declaration files.
13. Do not use ad-hoc `threading.Thread` calls outside of a declared cluster. All concurrency goes through `ClusterConnection`.
14. Do not hardcode confidence thresholds in negotiator code. All thresholds come from `clusters.yml`.
15. Do not bypass the import hook by importing app modules before `install_gospel_enforcement()` is called.
