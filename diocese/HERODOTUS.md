# Herodotus — Observation Architecture

**The invariant every construction in C_Brain must satisfy.**

Herodotus is the observation system of PyBrain. It is NOT logging. It is the
witness of morphisms — every arrow fired in C_Brain produces an observation
record routed through the Observation Trifecta. This document defines the
Observer producer interface, the Gateway entry point (Gospel IV), and the
end-to-end data flow from witnessed event to persistent triple.

---

## 1. Categorical Position

```
Pillar:           Life (4)
Construction:     Step 4 — Observation Product
Dependencies:     Resource (5) → Travel (6) → Life (4)
Parish:           life/observer, life/output, life/schema, life/instrument
Gospels:          IV (Gateway), XVI (Traced Arrow), XX (Trifecta), 3s (Rule of 3s)
```

Herodotus is the **Yoneda embedding** of C_Brain's morphism space: for every
morphism f: A -> B, the observation system records the representable functor
Hom(-, f) — capturing the full context of the arrow's invocation. This is
Gospel XVI: every traced arrow is observed, and observation preserves
compositional structure.

---

## 2. The Observation Trifecta (Gospel XX)

Every observation event produces to three streams with shared identifiers:

```
                   Observer Producer
                         |
                    [witness(event)]
                         |
                    RingBuffer (lock-free)
                         |
                    drain() -> enrich(session_id, sequence_id)
                         |
                  +------+------+
                  |      |      |
               Avro   Graph   Vector
               (fast   (struct  (search
                serial  capture) embed)
                record)
```

| Stream | Purpose | Substrate | Ref Doc |
|--------|---------|-----------|---------|
| **Avro** | Fast serial event record | fastavro -> `.avro` files | `docs/ref/fastavro.md` |
| **Graph** | Structural relationship capture | DuckDB graph extension | `docs/ref/duckdb/` |
| **Vector** | Searchable embedding | DuckDB HNSW index | `docs/ref/duckdb/`, `MATH.md` S3 |

### 2.1 Shared Identity

Every record across all three streams carries:

- **session_id** — UUID identifying the realm session (set once at realm boot)
- **sequence_id** — Monotonic 64-bit integer from `SequenceCounter` (cluster-wide leader)

The `(session_id, sequence_id)` pair is the **primary key** across the trifecta.
Any 2-of-3 projections suffice for fault tolerance (Rule of 3s).

### 2.2 Avro Event Schema

Defined in kitchen config (`kitchen/root/avro_schema.yml`), rendered by
`app/pillars/life/schema/avro_event.py:AvroEventSchema`:

```json
{
  "type": "record",
  "name": "<from config>",
  "namespace": "<from config>",
  "fields": [
    {"name": "session_id",  "type": "string"},
    {"name": "sequence_id", "type": "long"},
    {"name": "timestamp_ns","type": "long"},
    {"name": "source",      "type": "string"},
    {"name": "severity",    "type": "int"},
    {"name": "kind",        "type": "int"},
    {"name": "payload",     "type": ["null", "string"]}
  ]
}
```

Schema evolution follows fastavro's writer-schema/reader-schema resolution.
New fields MUST be nullable or have defaults. Backward compatibility is mandatory.

---

## 3. Observer Producer Interface

### 3.1 Design Principles

Observer producers are **shallow**: minimal import footprint, engrained at
module level. They replace the purged `get_logger`/`_log.debug("CALLED:...")`
antipattern (836 lines removed, 245 files, `app/logging/` deleted).

```
WRONG (purged antipattern):
    _log = get_logger(__name__)
    _log.debug("CALLED: some_function")

RIGHT (Observer producer):
    _observer = Observer("module_name")
    _observer.witness(WitnessInput(source="some_function", ...))
```

### 3.2 Observer Class

Defined at `app/pillars/life/observer.py`:

```python
class Observer(CategoricalEntity):
    """Witnesses arrows (morphisms) in C_Brain."""

    def __init__(self, name: str) -> None: ...
    def witness(self, event: WitnessInput | str) -> None: ...
    def attach_sink(self, sink: EventSink) -> None: ...
    def attach_cluster(self, cluster: WriterCluster) -> None: ...
    def set_session(self, session_id: str) -> None: ...
    def drain(self) -> list[dict]: ...
```

### 3.3 Module-Level Producer Pattern

Every module that produces observations instantiates at module level:

```python
"""my_module — does something.

Olog: MyThing -> SomePillar "transforms via"
"""
from app.pillars.life import Observer

_obs = Observer(__name__)


def my_function(inp: MyInput) -> MyOutput:
    """Gospel XV: 1 param, 1 return."""
    _obs.witness("my_function")
    # ... implementation ...
    return result
```

Properties:
- **Shallow**: Only imports `Observer` from `life` gateway
- **Module-level**: One observer per module, named by `__name__`
- **Minimal overhead**: `witness()` pushes to lock-free RingBuffer, returns immediately
- **No formatting**: No string interpolation at call site. Payload is optional.

### 3.4 WitnessInput — Structured Observation

```python
@dataclass(frozen=True)
class WitnessInput:
    """Product type for structured witness events (Gospel XV)."""
    source: str          # Fully-qualified function/method name
    kind: int            # Event kind enum value
    severity: int        # LogLevel enum value (not raw int)
    payload: str | None  # Optional structured payload (JSON string)
```

---

## 4. Instrument Decorators

### 4.1 @traced — Timing Instrument

Current implementation at `app/pillars/life/instrument/traced.py`.
Wraps function with monotonic timing. Future: emits WitnessInput with
ENTER/EXIT events and elapsed time.

### 4.2 @witnessed — Shallow Witness (Design Target)

From `background/autoposy/to_implement/example/usr/src/herodotus/instrument/decorators/witness.py`:

```python
def witnessed(observer_attr="_observer", severity=Severity.INFO):
    def decorator(fn):
        @wraps(fn)
        def wrapper(self, *args, **kwargs):
            observer = getattr(self, observer_attr, None) or get_observer()
            source = resolve_source(self, fn.__name__, fn.__qualname__)
            observer.witness(EventKind.ENTER, source, severity=severity)
            try:
                result = fn(self, *args, **kwargs)
            except BaseException as exc:
                observer.witness(EventKind.ERROR, source, payload=exc, severity=Severity.ERROR)
                raise
            else:
                observer.witness(EventKind.EXIT, source, severity=severity)
                return result
        return wrapper
    return decorator
```

Produces ENTER event at invocation, EXIT event on return, ERROR on exception.
Source resolution via `resolve_source()` using `__qualname__`. Severity.INFO
by default — lightweight enough for production.

### 4.3 @traced — Full Trace (Design Target)

From `background/autoposy/to_implement/example/usr/src/herodotus/instrument/decorators/traced.py`:

```python
def traced(observer_attr="_observer"):
    def decorator(fn):
        @wraps(fn)
        def wrapper(self, *args, **kwargs):
            observer = getattr(self, observer_attr, None) or get_observer()
            source = resolve_source(self, fn.__name__, fn.__qualname__)
            observer.witness(EventKind.ENTER, source,
                payload={"args": args, "kwargs": kwargs}, severity=Severity.DEBUG)
            try:
                result = fn(self, *args, **kwargs)
            except BaseException as exc:
                observer.witness(EventKind.ERROR, source,
                    payload={"exception": exc, "args": args, "kwargs": kwargs},
                    severity=Severity.ERROR)
                raise
            else:
                observer.witness(EventKind.EXIT, source,
                    payload={"result": result}, severity=Severity.DEBUG)
                return result
        return wrapper
    return decorator
```

Produces ENTER event with full args/kwargs, EXIT event with result payload.
Severity.DEBUG. Use for development and deep debugging only — heavier than
@witnessed.

### 4.4 Instrument Hierarchy

```
Instrument (abstract)
+-- @witnessed  — shallow ENTER/EXIT/ERROR, no payload, Severity.INFO
+-- @traced     — full ENTER/EXIT/ERROR with args/result payload, Severity.DEBUG
+-- @timed      — timing-only, elapsed measurement
+-- @metered    — metric collection (counters, gauges)
```

All instruments produce to the module-level `_obs: Observer` instance.

---

## 5. WriterCluster — Triple Dispatch (Rule of 3s)

Defined at `app/pillars/life/output/cluster.py`:

```
WriterCluster
+-- Writer[0]: AvroWriter     — fast serial record
+-- Writer[1]: GraphWriter    — structural relationship
+-- Writer[2]: VectorWriter   — searchable embedding
```

### 5.1 Dispatch Flow

```
Observer.drain()
    -> enrich with (session_id, sequence_id)
    -> WriterCluster.dispatch(DispatchInput)
        -> SequenceCounter.next_id()  [cluster-wide leader]
        -> fan_out to all 3 writers
        -> require min_success (2-of-3, from clusters.yml)
        -> failures -> DeadLetter queue (bounded capacity)
```

### 5.2 Quorum Semantics

- **min_success = 2**: At least 2 of 3 writers must succeed (Rule of 3s)
- Failed writes produce `DeadLetter` records (frozen dataclass)
- Dead letter queue has bounded capacity (from kitchen config)
- Quorum failure raises `ClusterException(E-4500)`
- Dead letter overflow raises `DeadLetterFullException(E-4530)`

### 5.3 Health Reporting

```python
@dataclass(frozen=True)
class HealthReport:
    total_dispatched: int
    avro_ok: bool
    graph_ok: bool
    vector_ok: bool
    dead_letters: int
```

---

## 6. Trace Artifacts (Design Target)

From `background/autoposy/to_implement/herodotus/trace/build.py`:

Per-function static semantic snapshot. Parses AST, compiles bytecode, resolves
code objects. Builds trace with:

| Component | What It Captures |
|-----------|-----------------|
| **Identity** | Module, qualname, content hash, ontology alignment (Gospel XIII) |
| **Bytecode** | Full disassembly of the code object |
| **Eval Frame** | CPython eval frame model (PYTHON.md S1) |
| **Native Pipeline** | Decode/execute/retire stages |
| **Memory Hierarchy** | L1 -> L2 -> L3 -> RAM mapping |
| **Error Propagation** | Expected exceptions with codes (Gospel XVII) |

Output: JSON trace files, one per function. These feed the Vector stream
(embeddings of function semantics) and the Graph stream (call graph edges).

---

## 7. Realm Avro Records (Design Target)

From `background/autoposy/to_implement/herodotus/instrument/realm_avro.py`:

Three special Avro records emitted during realm lifecycle:

| Record | When | Fields |
|--------|------|--------|
| **realm_metadata** | Realm creation | realm_id, manifest_hash, creation_ts |
| **session_end** | Session close | session_id, total_events, duration_ns |
| **observer_identity** | Observer registration | observer_name, module_path, pillar |

These bind the observation stream to the realm that produced it (Gospel VI:
Manifest Binding).

---

## 8. End-to-End Data Flow

```
                    MODULE CODE
                         |
                    _obs.witness(event)
                         |
                    RingBuffer.push(PushInput)
                         |  [lock-free, O(1)]
                         v
                    Observer.drain()
                         |
                    _ring_event_to_dict()
                         |
              +----------+-----------+
              |                      |
         EventSink.write()    WriterCluster.dispatch()
              |                      |
              v               SequenceCounter.next_id()
         [legacy path]              |
                              EnrichedRecord
                                    |
                          +---------+---------+
                          |         |         |
                    AvroWriter GraphWriter VectorWriter
                          |         |         |
                    .avro file  DuckDB     DuckDB
                                graph      HNSW
                                table      index
```

### 8.1 Kitchen Config Sources

All configuration flows from kitchen YAML (Gospel XII):

| Config File | Controls |
|-------------|----------|
| `avro_schema.yml` | Avro record schema (fields, types, namespace) |
| `clusters.yml` | WriterCluster settings (min_success, dead_letter_capacity) |
| `lfs_layout.yml` | Filesystem paths for output directories |
| `embedder.yml` | Embedding model, dimensions, batch size |

### 8.2 Filesystem Layout

Observation artifacts are written to the realm's FHS tree:

```
{realm_root}/
+-- var/
    +-- log/
    |   +-- events/          <- Avro event files
    +-- lib/
        +-- observers/       <- Observer state snapshots
        +-- state/           <- Sequence counter checkpoints
        +-- discovery/       <- Discovery artifacts
```

Managed by `app/pillars/location/platform/layout.py:RealmLayout`.

---

## 9. Implementation Inventory

### Built (Current State)

| Component | File | Status |
|-----------|------|--------|
| Observer | `app/pillars/life/observer.py` | Complete |
| RingBuffer | `app/pillars/life/core/ring_buffer.py` | Complete |
| WriterCluster | `app/pillars/life/output/cluster.py` | Complete |
| SequenceCounter | `app/pillars/life/output/sequence.py` | Complete |
| AvroEventSchema | `app/pillars/life/schema/avro_event.py` | Complete |
| AvroWriter | `app/pillars/life/output/avro_writer.py` | Complete |
| AvroHandler | `app/pillars/life/output/avro_handler.py` | Complete |
| EventSink | `app/pillars/life/output/event_sink.py` | Complete |
| MemorySink | `app/pillars/life/output/memory_sink.py` | Complete |
| DeadLetter | `app/pillars/life/output/cluster.py` | Complete |
| RecordAdapter | `app/pillars/life/output/record_adapter.py` | Complete |
| FlushEngine | `app/pillars/life/output/flush_engine.py` | Complete |
| Negotiator | `app/pillars/life/output/negotiator.py` | Complete |
| Embedder | `app/pillars/life/output/embedder.py` | Complete |
| Router | `app/pillars/life/output/router.py` | Complete |
| @traced (basic) | `app/pillars/life/instrument/traced.py` | Minimal |
| RealmLayout | `app/pillars/location/platform/layout.py` | Complete |

### Design Targets (in background/autoposy/to_implement/)

| Component | Source | Integration Target |
|-----------|--------|--------------------|
| @witnessed decorator | `example/usr/src/herodotus/instrument/decorators/witness.py` | `app/pillars/life/instrument/` |
| @traced (full) | `example/usr/src/herodotus/instrument/decorators/traced.py` | `app/pillars/life/instrument/` |
| Realm Avro records | `example/usr/src/herodotus/instrument/realm_avro.py` | `app/pillars/life/schema/` |
| Trace builder | `example/usr/src/herodotus/trace/build.py` | `app/pillars/life/` |
| Avro schema/write | `example/usr/src/herodotus/schema/` | `app/pillars/life/schema/` |
| Realm config | `example/usr/src/herodotus/instrument/realm_config.py` | `app/pillars/location/` |

---

## 10. Gospel Compliance Checklist

| Gospel | How Herodotus Satisfies It |
|--------|---------------------------|
| **I** (Process) | RingBuffer is lock-free; WriterCluster uses bounded queues |
| **IV** (Gateway) | Observer is THE single entry point for observation |
| **V** (Atomicity) | One file per domain: observer.py, cluster.py, avro_writer.py |
| **VIII** (Atomic Scope) | Each observation is atomic: push is O(1), dispatch is all-or-quorum |
| **XII** (Kitchen) | All config from YAML: schema, clusters, layout, embedder |
| **XIV** (Morphism) | Functions factored < 10 lines; drain/dispatch/fan_out separated |
| **XV** (Arrow) | All methods: 1 param (product type), 1 return |
| **XVI** (Observer) | THIS IS Gospel XVI. Observer producers replace get_logger |
| **XVII** (Error) | ClusterException(E-4500), DeadLetterFullException(E-4530) |
| **XX** (Trifecta) | Avro + Graph + Vector with shared session_id/sequence_id |
| **3s** (Rule of 3s) | 3 writers, 2-of-3 quorum, 3 output streams |
