# Pipeline — Ontological Mapping as Parish Transition

**Integration of the 6-stage pipeline into the pillar architecture, with
agent team composition at every stage.**

The pipeline in `docs/WIP/WORKFLOWS/TO_INTEGRATE=00/` is complete, executable
code implementing Gospel XIII (Ontological Mapping) + Gospel II (Absolute
Templating). This document specifies how each stage maps to the pillar
architecture via **Parish Transition** — not ad-hoc file moves, but formal
re-homing of morphisms from WIP staging to their categorical home.

---

## 1. Categorical Position

```
Crosscutting:     Gospel XIII + Gospel II
Construction:     Steps 3-7 (crosses Sight, Life, Communication)
Dependencies:     Resource -> Travel -> Location -> Life -> Sight -> Communication
Gospels:          II (Templating), III (Discovery), XIII (Olog), XXIII (Cluster)
```

The pipeline is a **composed functor**:

```
F_pipeline = F_output . F_generation . F_synthesis . F_projection . F_discovery

Where:
  F_discovery  : Artifact       -> DiscoveryProfile
  F_projection : DiscoveryProfile -> ProjectedModels
  F_synthesis  : ProjectedModels  -> LinkMLSchemas
  F_generation : LinkMLSchemas    -> PydanticModules
  F_output     : PydanticModules  -> ObservationTriple
```

Functoriality guarantees: each stage preserves identity and composition.
The pipeline composes because each stage's output type matches the next
stage's input type (Gospel XV: 1-in/1-out).

---

## 2. The Six Stages

### 2.1 Stage Map

| Stage | Focal Parish | WIP Source | Pillar Home | Olog Objects |
|-------|-------------|------------|-------------|-------------|
| **Discovery** | Sight/Transform | `discovery/` (4 files) | `app/pillars/sight/transform/discovery/` | DiscoveryProfile, PredicateStat |
| **Projection** | Sight/Transform | `projection/` (3 files) | `app/pillars/sight/transform/projection/` | DiscoveredNode, DiscoveredEdge, HierarchyEdge |
| **Synthesis** | Sight/Generate | `synthesis/` (2 files) | `app/pillars/sight/generate/synthesis/` | SynthesizedSchemas, LinkML YAML |
| **Generation** | Sight/Generate | `generation/` (2 files) | `app/pillars/sight/generate/codegen/` | GeneratedModules, Pydantic code |
| **Outputs** | Life/Output | `outputs/` (11 files) | `app/pillars/life/output/pipeline/` | Avro, CSV, Plotly, Dashboard |
| **Runtime** | Communication | `runtime/` (3 files) | `app/pillars/communication/orchestration/` | PipelineManifest, PipelineResult |

### 2.2 Parish Transition Protocol

For each stage, integration follows this protocol:

1. **Declare olog entries** in `mapping.yml` (Gospel XIII)
2. **Verify import hierarchy** — new code imports only from its pillar + dependencies (Gospel XVIII)
3. **Attach Observer producers** — every module gets `_obs = Observer(__name__)` (HERODOTUS.md)
4. **Validate Gospel compliance** — all 20 commandments, each file individually
5. **Write 1-to-1 tests** in `primative/tests/` mirroring source path (Gospel X)
6. **Update kitchen configs** — any hardcoded values -> YAML (Gospel XII)

---

## 3. Stage Details

### 3.1 Discovery

**What it does**: Parses input artifacts (RDF/Turtle via rdflib) and extracts
structural profile — classes, predicates, subclass relationships, graph stats.

**Key morphism**: `discover_graph_profile(input_paths) -> DiscoveryProfile`

| WIP File | Lines | Target File | Function |
|----------|-------|-------------|----------|
| `rdf_profile.py` | 330 | `sight/transform/discovery/rdf_profile.py` | Main discovery: parse TTL, extract classes/predicates/stats |
| `predicate_discovery.py` | 60 | `sight/transform/discovery/predicate.py` | Infer predicate patterns: URI/literal/blank, enum candidates |
| `rule_inference.py` | 28 | `sight/transform/discovery/rules.py` | Classify predicates: uri_only, enum, multivalued, numeric |
| `class_discovery.py` | 14 | `sight/transform/discovery/classes.py` | Sanitize class names from RDF URIs |

**Output type**: `DiscoveryProfile` (frozen dataclass, 22 fields)

### 3.2 Projection

**What it does**: Batch-converts discovered structures into typed Pydantic models.

**Key morphism**: `project_nodes_to_models(profile) -> list[DiscoveredNode]`

| WIP File | Lines | Target File | Function |
|----------|-------|-------------|----------|
| `graph_projector.py` | 86 | `sight/transform/projection/graph.py` | Node/edge model projection |
| `triple_projector.py` | 77 | `sight/transform/projection/triple.py` | Triple type projection (URI/Literal/Other) |
| `hierarchy_projector.py` | 8 | `sight/transform/projection/hierarchy.py` | Filter hierarchy edges |

**Output types**: `DiscoveredNode`, `DiscoveredEdge`, `HierarchyEdge`, `UriTriple`, `LiteralTriple`

### 3.3 Synthesis

**What it does**: Converts projected models into LinkML schemas (raw + optimized).

**Key morphism**: `synthesize_linkml_schemas(profile) -> SynthesizedSchemas`

| WIP File | Lines | Target File | Function |
|----------|-------|-------------|----------|
| `linkml_schema_builder.py` | 250+ | `sight/generate/synthesis/schema_builder.py` | THE SYNTHESIS ENGINE: LinkML classes, slots, enums, rules |
| `schema_postprocess.py` | 7 | `sight/generate/synthesis/postprocess.py` | Policy transforms on schema YAML |

**Output type**: `SynthesizedSchemas` (raw YAML + projected YAML)

### 3.4 Generation

**What it does**: Compiles LinkML schemas to Python code via PydanticGenerator.

**Key morphism**: `generate_pydantic_modules(schemas, output_dir) -> GeneratedModules`

| WIP File | Lines | Target File | Function |
|----------|-------|-------------|----------|
| `model_generator.py` | 80+ | `sight/generate/codegen/model_gen.py` | LinkML -> Pydantic via PydanticGenerator |
| `dynamic_importer.py` | 16 | `sight/generate/codegen/importer.py` | importlib dynamic loading of generated code |

**Output type**: `GeneratedModules` (schema YAMLs + generated .py files)

### 3.5 Outputs

**What it does**: Writes processed data to multiple output formats.

**Key morphism**: `write_all_visualizations(config) -> list[Path]`

| WIP File | Target File | Output Format |
|----------|-------------|--------------|
| `registry.py` | `life/output/pipeline/registry.py` | Output discovery + orchestration |
| `output_config.py` | `life/output/pipeline/config.py` | Split strategy, CURIE compression |
| `visualization_config.py` | `life/output/pipeline/viz_config.py` | Layout, filtering, styling presets |
| `csv_writer.py` | `life/output/pipeline/csv.py` | CSV triples/nodes/edges |
| `avro_writer.py` | `life/output/pipeline/avro.py` | Avro (Observation Trifecta) |
| `plotly_network_writer.py` | `life/output/pipeline/plotly_net.py` | 2D network visualization |
| `plotly_hierarchy_writer.py` | `life/output/pipeline/plotly_hier.py` | Hierarchy tree visualization |
| `hierarchy_tree_writer.py` | `life/output/pipeline/tree.py` | Filesystem tree output |
| `configurable_visualizer.py` | `life/output/pipeline/visualizer.py` | Configurable graph rendering |
| `dashboard_generator.py` | `life/output/pipeline/dashboard.py` | HTML dashboard linking artifacts |

### 3.6 Runtime

**What it does**: Orchestrates the full pipeline with caching and manifest tracking.

**Key morphism**: `run_dynamic_pipeline(input_paths, output_dir) -> PipelineResult`

| WIP File | Lines | Target File | Function |
|----------|-------|-------------|----------|
| `orchestrator.py` | 100+ | `communication/orchestration/pipeline.py` | THE ORCHESTRATOR: full pipeline execution |
| `manifest.py` | 36 | `communication/orchestration/manifest.py` | PipelineManifest + PipelineResult |
| `cache.py` | — | `communication/orchestration/cache.py` | Cache key generation + JSON persistence |

---

## 4. Pipeline Execution Flow

```
run_pipeline(input_paths, output_dir)
    |
    +-- [DISCOVERY] discover_graph_profile(ttl_files)
    |       RDF parsing -> DiscoveryProfile (22 fields)
    |
    +-- [SYNTHESIS] synthesize_linkml_schemas(profile)
    |       Generate raw + projected LinkML YAML
    |
    +-- [GENERATION] generate_pydantic_modules(schemas)
    |       LinkML -> Pydantic code via PydanticGenerator
    |
    +-- [GENERATION] load_generated_models()
    |       importlib dynamic import of generated code
    |
    +-- [PROJECTION] project_*_to_models()
    |       +-- project_triples -> DiscoveredTriple, UriTriple, LiteralTriple
    |       +-- project_nodes -> DiscoveredNode
    |       +-- project_edges -> DiscoveredEdge, HierarchyEdge
    |
    +-- [OUTPUTS] discover_enabled_outputs()
    |
    +-- [OUTPUTS] write_all_visualizations()
            +-- CSV, Avro, Plotly network/hierarchy
            +-- Hierarchy tree, configurable visualizer
            +-- dashboard_generator -> HTML dashboard
```

Output presets (from `pipeline.py`):
- **minimal** — CSV only
- **standard** — CSV + Plotly + dashboard
- **full_split** — All outputs with splitting by ontology/type/namespace
- **scalable** — Split at thresholds (100K rows, 100MB)

---

## 4a. Rendering Functors

Generation is not a single output — it is applying different functors to the same
source category. Each renderer is a functor from the knowledge graph category to
a target framework category:

```
Olog (source)
  |--- Render_FastAPI  : Olog → Cat_Python     (Python + Jinja2 + DuckDB)
  |--- Render_Spring   : Olog → Cat_Java       (Java + Thymeleaf + Cassandra)
  |--- Render_Next     : Olog → Cat_TypeScript  (TypeScript + React + Prisma)
  |--- Render_Flutter  : Olog → Cat_Dart       (Dart + Material + SQLite)
```

The template IS the functor (Gospel II: Absolute Templating). The knowledge graph
IS the source category. Generation IS functorial image computation. Functoriality
guarantees that structural relationships in the olog are preserved in every target:
if two entities compose in the olog, their generated implementations compose in the
target framework.

## 4b. Style as Natural Transformation

Once ontological structure is separated from rendering, style becomes a natural
transformation between rendering functors:

`α: Render_X ⇒ Render_X` — a parametric modification that changes colors,
typography, spacing, layout strategy across every generated artifact simultaneously.
The knowledge graph doesn't change. The functor parameters change. Every downstream
rendering updates coherently.

The naturality condition guarantees consistency: for any morphism f: A → B in the
olog, the diagram commutes:

```
Render_X(A) ---α_A---> Render_X(A)
    |                       |
 Render_X(f)            Render_X(f)
    |                       |
    v                       v
Render_X(B) ---α_B---> Render_X(B)
```

This is why the Trifecta matters: the three observation streams (Avro, Graph,
Vector) are three different functors on the same observation. Style transformations
between them are natural — the transformation commutes with composition.

---

## 5. Agent Team Architecture

### 5.1 Three-Layer Composition (Rule of 3s)

Every pipeline stage is executed by an agent team composed of three layers:

```
Layer 3: Cluster Quorum (Gospel XXIII)
    Ensures fault tolerance. 3 nodes per cluster, 2-of-3 success.
    No ad-hoc Thread(). Declared clusters only.
    |
Layer 2: Burr DAG (docs/ref/burr/)
    Orchestrates flow. State machines with typed transitions.
    Actions, persistence hooks, checkpoint/resume.
    |
Layer 1: Pydantic-AI Agents (docs/ref/pydantic-ai/)
    Do the work. Agent class with tools, structured output.
    Model-agnostic (Claude, GPT, local). Graph workflows.
```

### 5.2 Agent Roles per Stage

| Stage | Agent Role | What It Does | Burr DAG | Cluster |
|-------|-----------|-------------|----------|---------|
| **Discovery** | ANALYST | Parse artifacts, extract structure | `discover_dag` | 3 parallel parsers |
| **Projection** | ANALYST | Map structures to typed models | `project_dag` | 3 parallel projectors |
| **Synthesis** | CODE_GENERATOR | Build LinkML schemas from models | `synthesize_dag` | 3 schema builders |
| **Generation** | CODE_GENERATOR | Compile schemas to Python code | `generate_dag` | 3 code generators |
| **Outputs** | ORCHESTRATOR | Route data to output writers | `output_dag` | WriterCluster (built) |
| **Integration** | REVIEWER | Validate ontological alignment | `integrate_dag` | 3 validators |

Agent roles from `background/EXAMPLES/PYTHON/DJANGO/EXAMPLE=00/apps/agents/`:
- **ANALYST** — structural analysis, pattern recognition
- **CODE_GENERATOR** — schema compilation, code generation
- **REVIEWER** — validation, quality assessment
- **ORCHESTRATOR** — workflow coordination, routing

### 5.3 Burr DAG Pattern

Each stage DAG follows the same structure:

```python
@action(reads=["artifacts"], writes=["result"])
def process(state: State) -> State:
    """Single action in the DAG."""
    ...

dag = (
    ApplicationBuilder()
    .with_actions(validate=validate, process=process, emit=emit)
    .with_transitions(
        ("validate", "process", default),
        ("process", "emit", default),
        ("validate", "error", when(validation_failed)),
    )
    .with_state(artifacts=input_artifacts)
    .build()
)
```

### 5.4 Cluster Quorum per Stage

```
Stage Cluster (3 nodes)
+-- Node A: Primary agent instance
+-- Node B: Secondary agent instance
+-- Node C: Tertiary agent instance
    |
    +-- min_success = 2 (2-of-3 must agree)
    +-- Results compared for consistency
    +-- Divergent results -> REVIEWER agent arbitrates
```

This is distinct from the WriterCluster in HERODOTUS.md (which handles
observation output). Stage clusters handle **work execution** — ensuring
that agent processing is fault-tolerant.

---

## 6. Connection to Observation (HERODOTUS.md)

Every pipeline stage produces observations:

```
Stage Agent
    |
    +-- _obs.witness("stage.action")     [per-action observation]
    |
    +-- WriterCluster.dispatch()          [output stage only]
    |
    +-- PipelineManifest records          [runtime orchestrator]
```

The pipeline's `PipelineManifest` (frozen dataclass) captures:
- Schema IDs generated
- Modules produced
- Entities discovered
- Outputs enabled
- Artifact paths

This manifest IS the observation record for the pipeline run — it feeds
directly into the Graph and Vector streams.

---

## 7. Connection to Codegen (background/autoposy/to_implement/herodotus/codegen/)

The WIP pipeline's Generation stage connects to the Herodotus codegen pipeline:

```
WIP Pipeline                    Herodotus Codegen
-----------                    -----------------
model_generator.py             renderer.py (Jinja rendering)
  uses PydanticGenerator         uses manifest context
                               impl_manifest.py (ontology manifest)
                               impl_context.py (template context)
                               resolve_ontology_alignment.py
```

Both pipelines are implementations of Gospel II (Absolute Templating).
The WIP pipeline generates from LinkML schemas; the Herodotus codegen
generates from ontology manifests. They share the Jinja templating substrate
and should converge into a unified code generation system.

---

## 8. Integration Checklist

### Phase 1: Discovery + Projection (Sight Pillar)

- [ ] Create `app/pillars/sight/transform/discovery/` directory
- [ ] Create `app/pillars/sight/transform/projection/` directory
- [ ] Move 7 files with import hierarchy updates
- [ ] Add olog entries to `mapping.yml`
- [ ] Attach Observer producers to all modules
- [ ] Write 1-to-1 unit tests
- [ ] Update kitchen configs

### Phase 2: Synthesis + Generation (Sight Pillar)

- [ ] Create `app/pillars/sight/generate/synthesis/` directory
- [ ] Create `app/pillars/sight/generate/codegen/` directory
- [ ] Move 4 files with import hierarchy updates
- [ ] Add olog entries to `mapping.yml`
- [ ] Attach Observer producers
- [ ] Write 1-to-1 unit tests

### Phase 3: Outputs (Life Pillar)

- [ ] Create `app/pillars/life/output/pipeline/` directory
- [ ] Move 11 files with import hierarchy updates
- [ ] Connect to existing WriterCluster for Avro output
- [ ] Add olog entries to `mapping.yml`
- [ ] Write 1-to-1 unit tests

### Phase 4: Runtime (Communication Pillar)

- [ ] Create `app/pillars/communication/orchestration/` directory
- [ ] Move 3 files with import hierarchy updates
- [ ] Wire orchestrator to pillar-homed stages
- [ ] Add olog entries to `mapping.yml`
- [ ] Write 1-to-1 unit tests

### Phase 5: Agent Teams

- [ ] Define Burr DAGs per stage
- [ ] Implement Pydantic-AI agent roles
- [ ] Wire Cluster Quorum per stage
- [ ] Connect to Observation Trifecta

---

## 9. Top-Level Entry Points

| WIP File | Purpose | Integration Target |
|----------|---------|-------------------|
| `pipeline.py` | Public API: `run_pipeline()` | `app/pillars/communication/orchestration/api.py` |
| `model_codegen.py` | Standalone codegen | `app/pillars/sight/generate/codegen/standalone.py` |
| `schema_discovery.py` | Discovery utilities | `app/pillars/sight/transform/discovery/utils.py` |
| `validation.py` | Validation | `app/pillars/communication/orchestration/validation.py` |
| `visualizer.py` | Visualization interface | `app/pillars/life/output/pipeline/viz.py` |

---

## 10. Relationship to Other Diocese Documents

| Document | Relationship |
|----------|-------------|
| **HERODOTUS.md** | Pipeline stages produce to Observation Trifecta. WriterCluster handles output. |
| **REALM.md** | Pipeline executes within a Realm. Realm provides paths, session_id, artifacts. |
| **CONCEPTUAL.md** | Pipeline crosses construction steps 3-7 (Sight -> Life -> Communication). |
| **MATH.md** | Discovery/projection may process mathematical structures. Vector embeddings use S3. |
| **PYTHON.md** | Generation produces Python code. Cython hot paths (Gospel XXI) apply to pipeline. |
