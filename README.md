# weightless

Side-channel thought experiments on metadata, catalogs, and lakehouse-style architecture. The goal is to explore alternatives and question default assumptions—not to prescribe a design for any particular project.

**Design stance:** [Apache Gravitino](https://gravitino.apache.org/) is intentionally *federated first*—a unified control plane and API over many existing engines, catalogs, and silos. **weightless** explores the opposite emphasis: an *AI lakehouse first* design, where the primary constraints are AI-native workloads (agents, retrieval, training and inference pipelines, embeddings, and policy around model-driven access), and federation is a secondary concern rather than the organizing center.

This repository is intentionally **not** tied to the official design or roadmap of Apache Gravitino. Ideas here may contradict, simplify, or ignore constraints that matter in production systems; that is the point.

## Repository layout

| Path | Purpose |
| --- | --- |
| [`spec/`](spec/) | Canonical specifications: interfaces, data shapes, invariants, and acceptance criteria stable enough to implement against. |
| [`docs/adr/`](docs/adr/) | Architecture Decision Records: context, decision, consequences, and rejected alternatives. |
| [`notes/`](notes/) | Working notes and exploratory writing that may later promote into `spec/` or an ADR. |
| [`experiments/`](experiments/) | Small prototypes and spikes; disposable code tied to a narrow slice of a spec. |

**Key decision:** [ADR 0001 — AI lakehouse first as the design center](docs/adr/0001-ai-lakehouse-first-as-design-center.md).
