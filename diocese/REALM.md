# Realm — The Initial Object

**A Realm is an isolated execution domain from which all constructions are induced.**

The Realm is the categorical **initial object** of PyBrain's operational
architecture. Every session, every agent team, every observation stream, every
ontological integration originates from a Realm. There exists a unique morphism
from the Realm to every other operational construction — creation induces
ingestion, which induces agent teams, which induces integration.

---

## 1. Categorical Position

```
Pillar:           Location (2)
Construction:     Step 2 — Platform
Dependencies:     Resource (5) → Travel (6) → Location (2)
Parish:           location/platform/realm, location/platform/layout
Gospels:          VI (Manifest), XII (Kitchen), XIII (Olog), XXIII (Cluster)
```

### 1.1 Olog Declaration

```
a Realm
  has a Cookbook          "is defined by"
  has a RealmLayout      "structures filesystem at"
  has a SessionId        "identifies session via"
  has an AgentTeam       "launches"
  has an ObserverProducer "is observed by"

a Cookbook
  has a Manifest         "declares"
  has Ingredients        "requires"

Commutative diagram:
  Realm --has--> Cookbook --declares--> Manifest
  Realm --creates--> RealmLayout --populates--> FHS Tree
  The FHS Tree from RealmLayout = the paths declared in Manifest
```

---

## 2. Vision

From STORY.md:

> Each Realm is an isolated container defined by its own Cookbook (Manifest).
> A Realm has its own AbstractSpace (defining valid physics/logic) and Time
> reference. An Entity inside a "Financial Realm" cannot accidentally collide
> with an Entity in a "Physical Realm" because their Spaces are disjoint.

A Realm is what you create when you want PyBrain to **understand something**.
You point it at artifacts — a codebase, a dataset, a collection of documents —
and the Realm ingests, processes, and ontologically integrates those artifacts
into a knowledge graph.

---

## 3. The Primitive Interaction Loop

From `background/autoposy/snapshot=00/kitchen/proto.md`:

```
Template  defines the  Form.
Ingredient provides the Matter.
Recipe    provides the Process.
Seasoning provides the Context.
Cook      provides the Action.
```

Mapped to Realm operations:

| Kitchen Concept | Realm Mapping | What It Means |
|-----------------|---------------|---------------|
| **Template** | Cookbook manifest YAML | Defines what the Realm looks like |
| **Ingredient** | Input artifacts (code, data, docs) | Raw matter to be processed |
| **Recipe** | Pipeline stages (discovery -> integration) | How artifacts are transformed |
| **Seasoning** | Kitchen config (session_id, params) | Context for this specific run |
| **Cook** | Agent teams (Pydantic-AI + Burr + Cluster) | Who does the work |

---

## 4. Realm Lifecycle

### 4.1 Creation

```
create_realm(config: RealmConfig) -> Realm
    |
    +-- 1. Validate Cookbook manifest (Gospel VI)
    +-- 2. Generate realm_id (UUID) and session_id (UUID)
    +-- 3. Construct RealmLayout(LayoutConfig)
    +-- 4. RealmLayout.populate()  [mkdir FHS tree]
    +-- 5. Emit realm_metadata Avro record (HERODOTUS.md S7)
    +-- 6. Initialize Observer with session_id
    +-- 7. Return Realm (frozen, immutable after creation)
```

### 4.2 FHS Tree (built by RealmLayout)

```
{realm_root}/
+-- etc/                    <- Cookbook manifest, kitchen configs
+-- var/
|   +-- log/
|   |   +-- events/         <- Avro observation files
|   +-- lib/
|       +-- observers/      <- Observer state snapshots
|       +-- state/          <- Sequence counters, checkpoints
|       +-- discovery/      <- Discovery artifacts
+-- src/                    <- Ingested source artifacts
+-- gen/                    <- Generated code (LinkML -> Pydantic)
+-- out/                    <- Output artifacts (CSV, HTML, dashboards)
+-- graph/                  <- DuckDB graph database files
+-- vector/                 <- DuckDB HNSW index files
```

Managed by `app/pillars/location/platform/layout.py:RealmLayout`.
All paths from kitchen config (Gospel XII).

### 4.3 Ingestion

```
ingest(realm: Realm, sources: IngestInput) -> IngestResult
    |
    +-- 1. Classify source type (code, RDF/TTL, CSV, JSON, docs)
    +-- 2. Copy/sync artifacts to realm's src/ directory
    +-- 3. Register artifacts in realm manifest
    +-- 4. Emit ingestion observation events
    +-- 5. Return IngestResult (artifact inventory)
```

Source types and their handlers:

| Source Type | Handler | Pipeline Entry Point |
|-------------|---------|---------------------|
| RDF/Turtle | `rdf_profile.py` | Discovery stage |
| Python code | AST parser | Discovery stage (code structure) |
| CSV/JSON data | Schema inference | Projection stage |
| Markdown/docs | Text extraction | Projection stage (semantic) |
| Git repository | Clone + walk | Discovery stage (full codebase) |

### 4.4 Agent Team Launch

After ingestion, the Realm launches **agent teams** to process artifacts.
See `PIPELINE.md` Section 5 for the full agent team architecture.

```
launch_agents(realm: Realm, artifacts: IngestResult) -> AgentTeamHandle
    |
    +-- Layer 1: Pydantic-AI agents DO the work
    +-- Layer 2: Burr DAGs ORCHESTRATE the flow
    +-- Layer 3: Cluster Quorum ENSURES fault tolerance (Rule of 3s)
```

Each pipeline stage gets its own agent team. Teams are declared (Gospel XXIII:
no ad-hoc Thread()), composed via Burr DAG transitions, and observed via
Herodotus (HERODOTUS.md).

### 4.5 Integration

The terminal stage — processed artifacts become knowledge graph entries:

```
integrate(realm: Realm, processed: ProcessedArtifacts) -> IntegrationResult
    |
    +-- 1. Map processed artifacts to olog entries (Gospel XIII)
    +-- 2. Write to Graph stream (DuckDB graph tables)
    +-- 3. Write to Vector stream (DuckDB HNSW embeddings)
    +-- 4. Update realm manifest with integration metadata
    +-- 5. Emit integration observation events
```

### 4.6 Session Close

```
close_session(realm: Realm) -> None
    |
    +-- 1. Drain all observers
    +-- 2. Flush all writer clusters
    +-- 3. Emit session_end Avro record (HERODOTUS.md S7)
    +-- 4. Checkpoint sequence counters
    +-- 5. Close DuckDB connections
```

---

## 5. Realm as Categorical Initial Object

The universal property of the initial object: for every operational
construction X in C_Brain, there exists a **unique morphism** from Realm to X.

```
Realm ----!----> ObservationStream
  |                 (HERODOTUS.md: every realm has exactly one session)
  |
  +------!----> AgentTeam
  |                 (PIPELINE.md: realm config determines team composition)
  |
  +------!----> KnowledgeGraph
  |                 (Integration: realm artifacts map uniquely to graph entries)
  |
  +------!----> FederationPassport
                    (Future: realm identity uniquely determines federation creds)
```

The "!" denotes uniqueness — there is exactly one way to construct each
dependent from a given Realm configuration. This is enforced by:

- **Manifest Binding** (Gospel VI): Cookbook determines structure
- **Kitchen Config** (Gospel XII): No hardcoded values, all from config
- **Observation Invariant** (HERODOTUS.md): Session ID set once at creation

### 5.1 Terminal Object and Zero Object Collapse

The Realm is the initial object — unique morphism to every construction. The
fully integrated knowledge graph is the **terminal object** — every artifact
flows into it. There exists a unique morphism from every construction to the
knowledge graph (integration maps all processed artifacts into olog entries).

Once the system can ingest its own output (see STORY.md: The Fixed Point),
initial and terminal collapse into the same construction. The knowledge graph
is simultaneously where artifacts go to be understood and where new artifacts
come from. This is a **zero object**: both initial and terminal.

A category with a zero object has a distinct character. Every pair of objects
has a **zero morphism** (factor through the zero object). This means: between
any two artifacts, there exists a canonical "do nothing" transformation that
goes through the knowledge graph. This is the foundation of compositionality —
any two artifacts can be composed by factoring through the shared representation.

```
Realm (initial) ----!----> KnowledgeGraph (terminal)
     ^                          |
     |                          |
     +------re-ingest-----------+
                  (collapse: initial = terminal = zero object)
```

The zero object emerges at the fixed point — when the Realm that produces
the knowledge graph IS the knowledge graph that produces the Realm. Until
then, initial and terminal are distinct, and the system is an open pipeline.

---

## 6. Federation (Future — Phase 22+)

When Realm A wants to communicate with Realm B:

```
Realm A                        Realm B
   |                              |
   +-- serialize Passport  -->  Translator
        (AbstractValue)            |
                              validate against
                              Realm B's manifest
                                   |
                              accept/reject
```

- **Passport**: Serialized AbstractValue carrying realm identity + data
- **Translator**: Validates Passport against receiving realm's manifest
- **AbstractTranslator**: `app/pillars/communication/` (18-line stub)
- Spaces are disjoint: entities cannot collide across realms

---

## 7. Implementation Inventory

### Built

| Component | File | Status |
|-----------|------|--------|
| RealmPlatform (ABC) | `app/pillars/location/platform/realm.py` | Stub (18 lines) |
| RealmLayout | `app/pillars/location/platform/layout.py` | **Complete** (74 lines) |
| LayoutConfig | `app/pillars/location/platform/layout.py` | Complete |

### Design Targets

| Component | Source | Notes |
|-----------|--------|-------|
| RealmConfig | `background/autoposy/to_implement/herodotus/config/realm_config.py` | Frozen dataclass for kitchen/root.yml |
| Realm Avro records | `background/autoposy/to_implement/herodotus/instrument/realm_avro.py` | Metadata, session_end, observer_identity |
| Realm lifecycle | This document (S4) | Full create -> ingest -> launch -> integrate -> close |
| Cookbook engine | `background/autoposy/snapshot=00/kitchen/proto.md` | Template/Ingredient/Recipe/Seasoning/Cook |

### Not Built

| Component | Blocks | Priority |
|-----------|--------|----------|
| Realm creation engine | Everything downstream | Critical |
| Ingestion handlers | Pipeline execution | Critical |
| Agent team launcher | Processing | Critical |
| Knowledge integration | Output quality | Critical |
| Federation protocol | Cross-realm | Phase 22+ |

---

## 8. RealmConfig — Product Type

From `background/autoposy/to_implement/example/usr/src/herodotus/instrument/realm_config.py`:

```python
@dataclass(frozen=True)
class RealmConfig:
    """Parsed kitchen/root/root.yml Realm configuration.
    All paths resolved to absolute at parse time."""

    name: str                       # Realm name (required)
    global_log: bool                # Enable global logging
    description: str                # Human-readable description
    location_file: str              # Absolute path to location config
    global_file: str                # Absolute path to global registry
    codec: str                      # Avro codec (deflate, snappy, etc.)
    global_sync_events: bool        # Sync events to global
    global_sync_discovery: bool     # Sync discovery artifacts to global
    global_sync_realm_metadata: bool # Sync realm metadata to global
    init_sync: bool                 # Sync on init
    init_global_sync: bool          # Global sync on init
    post_sync: bool                 # Sync on session close
    post_cleanup_sessions: bool     # Clean up sessions on close
    post_cleanup_realm: bool        # Clean up realm on close

    @classmethod
    def from_yaml(cls, path: Path) -> RealmConfig:
        """Load and parse kitchen root.yml. Resolves paths to absolute."""
        ...
```

Loaded from `kitchen/root.yml` per Gospel XII. Paths resolved to absolute
from CWD at parse time. Missing required fields (`name`, `location.file`,
`global.file`) raise `RealmConfigException`.

---

## 9. Relationship to Other Diocese Documents

| Document | Relationship |
|----------|-------------|
| **HERODOTUS.md** | Realm lifecycle events are observed per HERODOTUS.md. Session ID originates here. |
| **PIPELINE.md** | Pipeline stages execute within a Realm. Realm provides filesystem paths + config. |
| **CONCEPTUAL.md** | Realm is a construction in Step 2 (Location pillar) of the 6-Pillar architecture. |
| **STORY.md** | Realm is the "Federated Realm" described in the vision narrative. |
| **MATH.md** | Realm ingests artifacts that may include mathematical structures (S1-S7). |
