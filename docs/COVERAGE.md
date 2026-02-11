# Documentation Coverage Checklist (No-holes)

이 문서는 “문서셋에 빵꾸가 없는지”를 점검하는 체크리스트다.  
각 항목은 반드시 **어딘가의 최종 권위 문서**로 연결되어야 한다.

## Big Picture

- Canon entrypoint: `../rfc/005`
- Doc index: `./INDEX.md`

## Core Domains (SSOT)

- Definition Authority (RDS): `spec/api-control-plane.md`, `spec/storage-layout.md`
- Execution Authority (Dynamo): `spec/execution-semantics.md`, `spec/api-execution-plane.md`
- Raw (S3 + RawIndex): `spec/raw-and-replay-index.md`
- Lineage Authority: `spec/lineage-graph.md` + ADR `adr/0003-lineage-store-selection.md`
- Egress Authority: `spec/admission-and-egress.md`

## Determinism / Idempotency / Replay

- Determinism contract + fingerprint: `spec/determinism-and-fingerprint.md`
- Replay lookup rules: `spec/raw-and-replay-index.md`
- Runbook: `runbooks/replay-rerun-requeue.md`

## Workers / Commit Gate

- Worker protocol: `spec/worker-protocol.md`
- Commit gate semantics: `spec/execution-semantics.md`

## Validation / Governance

- Workflow validation & lint: `spec/workflow-validation-and-lint.md`
- Policy system: `spec/policy-system.md`
- Cost governance runbook: `runbooks/cost-governance.md`

## Events / Projections / Consistency

- Authority & events: `spec/authority-and-events.md`
- Outbox decision: `adr/0002-outbox-must.md`

## Security / Compliance

- Security overview: `security/README.md`
- Effective delete decision: `adr/0004-compliance-effective-delete.md`
- Compliance spec: `spec/compliance-and-retention.md`

## Observability / Testing / Deployment

- Observability: `observability/README.md`
- Testing matrix: `testing/README.md`
- Deployment guide: `deployment/README.md`
- CI guide: `ci/README.md`

