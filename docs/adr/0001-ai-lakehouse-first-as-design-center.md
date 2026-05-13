# 1. AI lakehouse first as the design center

## Status

Accepted

## Context

Data platforms differ in what they optimize first. A *federated-first* system starts from the problem of unifying many existing engines, catalogs, and silos behind a single control plane and API. That ordering is appropriate when the primary pain is visibility, governance, and interoperability across heterogeneity.

**weightless** is an exploratory space (not the [Apache Gravitino](https://gravitino.apache.org/) roadmap). Here we intentionally invert the emphasis: we want to reason about metadata, access, and lifecycle when the *dominant* integrators are AI-native workloads rather than human SQL sessions alone.

The term **AI lakehouse** is not standardized in the industry. Without a working definition, discussions drift into slogans. This ADR fixes vocabulary and the first-order design question for this repository.

## Decision

1. **Primary design center:** Treat **AI lakehouse first** as the organizing constraint for ideas and specs in this repo: model and agent data paths, retrieval, embeddings, training and inference pipelines, and policy/audit for model-driven access to lakehouse-resident data.

2. **Secondary concern:** **Federation** (unifying many external catalogs and engines) is explicitly *not* the first lens. It may appear later as a bridge, but it does not set the default abstractions.

3. **Working definition — AI lakehouse:** An architecture on an **open table format + object storage** lakehouse base that co-designs **tabular, semi-structured, and unstructured data** with **AI assets and access paths**—for example embedding indexes, feature tables, dataset versioning, and stable schemas/contracts exposed to tools and agents—under **shared storage, metadata, and governance semantics**. The default integration surface and evolution pressure come from **models, agents, and ML pipelines**, not from “connect every external catalog first.” Durable artifacts that would normally live in auxiliary services (catalog rows, manifests, small indexes, audit logs) should be **materialized in the object namespace** or carried in **process-local/ephemeral** components, not in additional *required* external stores—see (5).

4. **Distinction from a classic data lakehouse:** A classic lakehouse often centers **interactive SQL analytics, table lifecycle, and batch/stream processing**. An AI lakehouse still values SQL, but optimizes first for **repeatable model inputs/outputs, retrieval, and production/experiment parity** across the same governed data.

5. **Object storage as the only mandatory external dependency:** The **only** hard external infrastructure dependency we introduce in this design line is **object storage** (S3-compatible API) for durable state. We do **not** mandate other managed services (vendor databases, buses, proprietary vector DBs, hosted metadata planes, etc.) as part of the core platform closure. **AI workload breadth** is pursued by **formats, layouts, protocols, and optional adapters** on top of that substrate—including, where useful, *optional* integrations that operators may attach—without making those integrations **required** to run the core story.

## Consequences

- Specs and experiments should state **which AI workload** they serve (e.g. batch training, online inference, RAG, agent tool calls) before debating multi-catalog federation.

- Interfaces may prefer **machine-oriented contracts** (schemas, events, policy tokens) when that clarifies agent and pipeline usage—even if that is not the shortest path for ad hoc human exploration.

- Some federated scenarios will be intentionally **out of scope** for early slices; that is expected, not a bug.

- Designs should show **where bits land in the object namespace** (paths, manifests, table metadata, sidecar indexes) when introducing a capability that would typically imply another service.

- “As much AI workload as possible” is a **product of the data plane and contracts**, not of expanding the mandatory dependency graph.

## Non-goals (for early material in this repo)

- Claiming a universal replacement for federated metadata systems.

- Full compatibility stories across every engine and legacy catalog unless an ADR explicitly expands scope.

- A core design that **requires** a separate OLTP database, message broker, or SaaS vector store to exist before the system is coherent—those may appear as **optional** adapters, not as pillars of the minimal closure.

## Alternatives considered

- **Federated first:** Matches common enterprise reality and products such as Apache Gravitino; rejected *as the organizing center for this repo* because it is already well explored elsewhere and would duplicate the default framing we are trying to step away from.

- **SQL-only lakehouse first:** Simpler and familiar; rejected as the *sole* center because it under-specifies agent, retrieval, and model-governed access paths that we want to foreground.

- **Best-of-breed services first:** Adds Kafka, cloud catalog, managed vector DB, etc. as assumed dependencies; rejected for the **core** envelope here because it improves time-to-demo at the cost of the strict object-storage-only dependency rule we want to explore.

## Vocabulary and comparisons

The following distinctions are **not** universal industry definitions; they are the working vocabulary for specs in this repository so we do not conflate layers.

### Schema vs namespace

| | **Namespace** | **Schema** |
| --- | --- | --- |
| **Addresses** | Naming and isolation: *where* something lives, *what* logical name it has under a hierarchy, and boundary of visibility or ownership. | Shape and semantics: *what* the records look like—fields, types, nullability, nested structures, and evolution rules. |
| **Typical locus** | Catalog / database–style hierarchy; URI or bucket prefixes; tenant-scoped path conventions. | Table- (or message-) level column definitions; formats express schema (e.g. Parquet, Iceberg schema). |
| **Evolution** | Renames, moves, delegation of authority (“which prefix belongs to which logical name”). | Reader/writer compatibility when columns or types change. |

On object storage alone, **prefix + convention** often implements namespace; **schema** is carried by file metadata and table-format metadata trees.

### Fileset vs Lance

| | **Fileset** (e.g. a fileset-style abstraction in catalog products) | **Lance** (format) |
| --- | --- | --- |
| **Layer** | Logical asset: a **named collection of files** as a first-class unit (paths, lifecycle, policy). | Storage **format**: columnar layout optimized for ML-oriented access patterns (e.g. vectors, selective reads). |
| **Says** | “These paths belong to one **dataset slice**.” | “On disk, **columns and vectors** are encoded and indexed **this way**.” |

A fileset can **contain** Lance files; they are orthogonal dimensions—**bundle of paths** vs **encoding of bytes**.

### Generic table format abstraction

**Generic table format** means an API or spec that stays **as independent as possible** of a single implementation (Delta Lake, Apache Iceberg, Apache Hudi, …), relying only on shared notions: snapshots, schema, partitioning, commits, and a stated consistency model—or a thin protocol for manifests and object layout.

Trade-offs: either a **lowest common denominator** of features, or an explicit **capability matrix** (e.g. branch/time travel unavailable unless the backing format supports it). Under the **object-storage-only** envelope here, “generic” also implies **metadata and pointers** must be placeable in the object namespace rather than assuming a proprietary hosted metastore—unless introduced as an **optional** adapter.

### Model management

**Model management** is the lifecycle of **ML artifacts**, not table schema alone: versioned weights and configs, lineage to **data snapshots / feature views / eval sets**, promotion stages (dev → prod), rollback, and audit or deployment policy.

Artifacts still **land in object storage** in this design line; registry-like metadata is either **manifests in the object namespace** or **optional** external registries—never a mandatory extra store for the minimal story.

### Table-format operations: Delta Lake, Lance, Apache Iceberg

“Management” here means **operations beyond raw Parquet I/O**: transactional metadata, maintenance, and layout—not only reads/writes.

| | **Delta Lake** | **Apache Iceberg** | **Lance** |
| --- | --- | --- | --- |
| **Core metadata object** | Transaction **log** under `_delta_log` (JSON), defining table versions. | **Catalog pointer** to metadata files, snapshots, manifests, and schema evolution. | **Dataset** versioning and columnar/vector-oriented **physical** layout and indexes. |
| **Typical maintenance** | OPTIMIZE, Z-order, VACUUM, time travel via log versions. | Snapshot expiration, manifest merging, partition evolution, branches/tags where supported. | Compaction, index build, version-oriented maintenance tuned for ML/random access patterns. |
| **Emphasis** | Strong in **SQL / BI / streaming** integration paths. | Broad **open-table** interoperability across engines. | **AI-native** column store; not a drop-in replacement for the full **federated catalog** story of Iceberg. |

All three can run with **object storage** as the durability layer; Iceberg and Delta still assume some **catalog binding** (even if minimal—a pointer to current metadata in object storage). Specs should state that **minimal catalog closure** explicitly.

## References

- Project README: design stance and repository layout.
