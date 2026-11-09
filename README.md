<p align="center"><img src="readme-banner.svg" alt="Crawler Engine / Design — Workflow authority, deterministic replay, lineage, and operating contracts." width="100%"></p>

# Wine Crawler Engine Design

A **design and operating-contract repository** for a collection engine. It connects RFC rationale to implementation specifications, operational runbooks, and architecture decisions. It is distinct from the runnable crawler application.


## Engineering highlights

| Mechanism | Engineering purpose |
| :--- | :--- |
| **Authority / event / projection split** | The design assigns one authority store to each domain and connects domains through events and projections rather than cross-store transactions. |
| **Explicit execution identity** | Run, task, attempt, artifact, idempotency, and lease concepts have separate execution contracts. |
| **Immutable raw capture and replay** | Raw metadata selection and immutable blob storage are specified separately from derived artifacts. |
| **Lineage as a first-class model** | Artifact nodes and transform edges make derived outputs traceable to their inputs. |
| **Operational failure contracts** | Runbooks cover replay, schema drift, backfill, egress incidents, cost governance, and retention/compliance. |


## Architecture at a glance

```mermaid
flowchart LR
    D["Definition authority: workflows and versions"] --> E["Domain events and projections"]
    E --> X["Execution authority: runs, tasks, attempts"]
    X --> R["Raw index and immutable blobs"]
    R --> L["Lineage and derived artifacts"]
    P["Admission and egress policy"] --> X
    O["Runbooks and recovery contracts"] -.-> X
```

## Reading order

1. [Canonical RFC](rfc/005): current design and ownership split.
2. [Documentation index](docs/INDEX.md): the implementation and operations map.
3. [Specifications](docs/spec): execution states, events, and storage contracts.
4. [Runbooks](docs/runbooks): replay, recovery, backfill, and cost management.
5. [Architecture decisions](docs/adr): focused decisions and their rationale.

## Core contracts

| Contract | Problem addressed |
| --- | --- |
| Execution semantics | Run, Task, Attempt, and Artifact states and transition guards |
| Determinism and idempotency | Canonicalization, fingerprints, and replay rules |
| Authority and projection | Ownership of authoritative state, read projections, and events |
| Raw lookup and replay | RawIndex and replay lookup behavior |
| Resource and egress control | Admission, capacity, and external-transfer costs |

## Maintaining the design

Read `rfc/005` as the current canon. Earlier stubs and archived RFCs retain design history. A documented contract is not evidence that its implementation, deployment, or operating qualification has completed.

- [Documentation guide](docs/README.md)
- [Coverage checklist](docs/COVERAGE.md)
- [Roadmap](docs/ROADMAP.md)
- [Document templates](docs/templates)
