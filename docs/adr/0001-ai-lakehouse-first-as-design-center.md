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

3. **Working definition — AI lakehouse:** An architecture on an **open table format + object storage** lakehouse base that co-designs **tabular, semi-structured, and unstructured data** with **AI assets and access paths**—for example embedding indexes, feature tables, dataset versioning, and stable schemas/contracts exposed to tools and agents—under **shared storage, metadata, and governance semantics**. The default integration surface and evolution pressure come from **models, agents, and ML pipelines**, not from “connect every external catalog first.”

4. **Distinction from a classic data lakehouse:** A classic lakehouse often centers **interactive SQL analytics, table lifecycle, and batch/stream processing**. An AI lakehouse still values SQL, but optimizes first for **repeatable model inputs/outputs, retrieval, and production/experiment parity** across the same governed data.

## Consequences

- Specs and experiments should state **which AI workload** they serve (e.g. batch training, online inference, RAG, agent tool calls) before debating multi-catalog federation.

- Interfaces may prefer **machine-oriented contracts** (schemas, events, policy tokens) when that clarifies agent and pipeline usage—even if that is not the shortest path for ad hoc human exploration.

- Some federated scenarios will be intentionally **out of scope** for early slices; that is expected, not a bug.

## Non-goals (for early material in this repo)

- Claiming a universal replacement for federated metadata systems.

- Full compatibility stories across every engine and legacy catalog unless an ADR explicitly expands scope.

## Alternatives considered

- **Federated first:** Matches common enterprise reality and products such as Apache Gravitino; rejected *as the organizing center for this repo* because it is already well explored elsewhere and would duplicate the default framing we are trying to step away from.

- **SQL-only lakehouse first:** Simpler and familiar; rejected as the *sole* center because it under-specifies agent, retrieval, and model-governed access paths that we want to foreground.

## References

- Project README: design stance and repository layout.
