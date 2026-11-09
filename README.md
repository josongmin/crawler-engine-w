# Wine Crawler Engine Design

A **design and operating-contract repository** for a collection engine. It connects RFC rationale to implementation specifications, operational runbooks, and architecture decisions. It is distinct from the runnable crawler application.

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
