# ADRs

작은 결정(키 포함 여부, outbox MUST 승격, lineage store 선택 등)은 ADR로 기록한다.

## Template

- `../templates/ADR_TEMPLATE.md`

## Suggested ADR Backlog

- ADR: Architecture Decision Map (메타 ADR)
- ADR: Idempotency Key Scope (`workflow_version` 포함 여부)
- ADR: Outbox Pattern MUST 여부(Dispatch/Enqueue 경로)
- ADR: Lineage Store 선택(GraphDB vs RDB vs Dynamo)
- ADR: Compliance “Effective Delete” 전략(불변 원칙 유지)

