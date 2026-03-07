# The Story of PyBrain

**A Distributed Operating System for Reality.**

---

## The Core Idea

Everything exists as a thing:

1. **Who** — Identity (Resource pillar)
2. **What** — Classification (Travel pillar)
3. **Where** — Position (Location pillar)
4. **How** — Transformation (Sight pillar)
5. **With what** — Observation (Life pillar)
6. **Why** — Communication (Communication pillar)

These six questions map to the six pillars of C_Brain. Any entity in any domain
can be described by answering all six. The system that answers them automatically,
for any input, is PyBrain.

---

## What PyBrain Does

You point PyBrain at something — a codebase, a dataset, a collection of
documents, an API — and it **understands it**.

```
YOU HAVE ARTIFACTS
    |
    v
CREATE A REALM
    A Realm is an isolated container. It has its own filesystem,
    its own observation stream, its own knowledge graph.
    Think of it as a sandbox with a brain.
    |
    v
INGEST ARTIFACTS INTO THE REALM
    Download, sync, copy. Artifacts land in the Realm's src/ directory.
    The Realm now has raw matter — the Ingredients of the Kitchen.
    |
    v
LAUNCH AGENT TEAMS
    Three layers, following the Rule of 3s:

    Layer 1: Pydantic-AI agents DO the work
             Analysts parse structure. Generators build schemas.
             Reviewers validate quality. Orchestrators route flow.

    Layer 2: Burr DAGs ORCHESTRATE the flow
             State machines with typed transitions. Checkpoint/resume.
             Each pipeline stage is a DAG action.

    Layer 3: Cluster Quorum ENSURES fault tolerance
             3 nodes per stage. 2-of-3 must succeed.
             No single agent failure kills the pipeline.
    |
    v
PROCESS THROUGH THE PIPELINE
    discovery   -> What structures exist in the artifacts?
    projection  -> How do those structures map to typed models?
    synthesis   -> What LinkML schemas describe those models?
    generation  -> What Python code implements those schemas?
    outputs     -> Avro records, graph entries, vector embeddings,
                   CSV exports, Plotly visualizations, HTML dashboards
    |
    v
ONTOLOGICALLY INTEGRATE
    Every processed artifact maps to an olog entry (Gospel XIII).
    The knowledge graph grows. Vector embeddings enable semantic search.
    Graph relationships enable structural queries.
    The Realm now UNDERSTANDS the artifacts.
```

---

## The Architecture

### The Realm (The Sandbox)

Each Realm is an isolated container defined by its own Cookbook (Manifest).
A Realm has its own AbstractSpace (defining valid physics/logic) and Time
reference. An Entity inside a "Financial Realm" cannot accidentally collide
with an Entity in a "Physical Realm" because their Spaces are disjoint.

The Realm runs an internal loop — the Mechanisms layer — that constantly
tries to optimize the state of its Graph: maximize throughput, minimize
latency, balance accounts, improve ontological coverage.

See `REALM.md` for the full lifecycle specification.

### The Graph (The State)

- **Nodes**: Entities (Resources, Users, Structures, Artifacts)
- **Edges**: Relationships or Flows ("Owns", "Transfers To", "Controls", "Is-A")
- **Metrics**: The AbstractValues attached to Nodes/Edges are not static —
  they are streams of data collected over time via the Observation Trifecta

### The Observation (The Witness)

Herodotus witnesses every morphism. Every function call, every transformation,
every agent action produces an observation record routed through three streams:

- **Avro** — fast serial event record
- **Graph** — structural relationship capture
- **Vector** — searchable embedding

2-of-3 suffices for fault tolerance. Shared `(session_id, sequence_id)` across
all three. See `HERODOTUS.md` for the full observation architecture.

### The Federation (The Interconnect)

When Realm A wants to talk to Realm B, it doesn't just send raw data. It sends
a **Passport** — a serialized AbstractValue carrying realm identity and data.
The **Translator** at the edge of Realm B validates this Passport against
Realm B's manifest.

This is the GraphAtlas vision: federated ontological infrastructure where
independent domain ontologies interoperate through explicit, structure-preserving
mappings (functors). See `docs/slop/HIGH.md` for the external formulation.

---

## The Kitchen (How Things Get Cooked)

From `background/autoposy/snapshot=00/kitchen/proto.md`:

| Concept | Maps To | Meaning |
|---------|---------|---------|
| **Template** | Cookbook manifest YAML | Defines what the Realm looks like |
| **Ingredient** | Input artifacts | Raw matter to be processed |
| **Recipe** | Pipeline stages | How artifacts are transformed |
| **Seasoning** | Kitchen config | Context for this specific run |
| **Cook** | Agent teams | Who does the work |

The dynamic templating engine (Gospel II) allows code generation on the fly
during artifact processing. Templates define Forms. Recipes define Processes.
The Cook executes the Recipe with Ingredients and Seasoning to produce output.

---

## The Pillar DAG

```
         Communication (1)
          ^    ^    ^
     +----+    |    +----+
     |    Sight (3)      |
     |    ^    ^         |
     | +--+    +--+      |
     | Location(2) Life(4)
     | ^    ^      ^
     | +--+ +--+ +-+
     |    Travel (6)
     |       ^
     +-------+
        Resource (5)
```

Construction order follows the DAG bottom-up. Resource is the foundation.
Communication is the colimit that unifies all pillars. See `CONCEPTUAL.md`
for the full categorical construction.

---

## The Diamond

Everything is an AbstractThing:

```
AbstractThing(ABC)                   <- primordial root
+-- Thing(AbstractThing)             <- @dataclass(frozen=True)
+-- AbstractType(AbstractThing, Enum) <- type enumerations
+-- AbstractMechanism(AbstractThing) <- transform engines
```

Frozen dataclasses everywhere. No bare dict. Identity inherited. Every class
maps to an ontological entry. See `CONCEPTUAL.md` Part 0 for the category
C_Brain.

---

## The Gospels

20 absolute commandments govern every file, every function, every edit.
No exceptions. The Gospels are the categorical invariants of C_Brain —
violating one means the construction is ill-formed.

See `CONCEPTUAL.md` Part 3 for the full Gospel canon.

---

## The Fixed Point

The ludicrous capability threshold: when the pipeline can ingest *its own output*.
When PyBrain generates a frontend, then ingests that generated frontend back through
the pipeline, and the resulting olog entries are *isomorphic* to the original entries
that generated it — that's the fixed point. That's the moment the system becomes
self-describing in the Yoneda sense: the representation of the thing IS the thing,
up to natural isomorphism.

```
generate → observe → re-ingest → refine → regenerate
                    ↑                          |
                    └──────────────────────────┘
```

Each cycle enriches the knowledge graph with observations about how artifacts
actually behaved. The Rule of 3s at the agent layer (Pydantic-AI does / Burr
orchestrates / Cluster Quorum ensures) combined with Rule of 3s at the observation
layer (Avro / Graph / Vector) means every cycle is both fault-tolerant and fully
observed.

The fixed point is not a single moment — it is a convergence. Each iteration
produces artifacts that, when re-ingested, yield olog entries closer to the
entries that generated them. Convergence means the system has fully understood
both the domain and its own representation of that domain.

---

## Where We Are

- 899 Python files, 2659 tests, ~3350 manifests
- 6-pillar architecture built through construction step 4
- Ontological mapping operational (391 root entries, MRO resolution)
- Observation Trifecta partially built (RingBuffer, WriterCluster, AvroWriter)
- Complete pipeline code in WIP (5,375+ lines, ready for integration)
- RealmLayout complete (FHS directory manager)
- Math library scaffolded (54 files, 7 sections matching MATH.md)

## What Comes Next

1. **HERODOTUS.md** specifies the observation invariant (done)
2. **REALM.md** specifies the initial object (done)
3. **PIPELINE.md** specifies integration via Parish Transition (done)
4. Realm creation engine implementation
5. Agent team orchestration
6. Pipeline integration into pillar architecture
7. Knowledge integration (terminal object)
8. Federation protocol (Phase 22+)

---

## Diocese Documents

| Document | What It Specifies |
|----------|------------------|
| **STORY.md** | This document — the narrative |
| **HERODOTUS.md** | Observation architecture (the invariant) |
| **REALM.md** | Realm lifecycle (the initial object) |
| **PIPELINE.md** | Pipeline integration (the composition law) |
| **CONCEPTUAL.md** | 6-Pillar categorical architecture |
| **MATH.md** | Mathematical foundations (7 sections) |
| **PYTHON.md** | CPython internals |
| **MARATHON_GOAL_00.md** | Keyboard-to-silicon exec trace |
| **MYSTERY_VAN.md** | Ontological mapping directive |