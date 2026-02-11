# Documentation & Implementation Roadmap (Coverage-driven)

이 로드맵은 “기능 추가 순서”가 아니라 **장기 운영에서 논쟁을 끝내는 SSOT 문서 커버리지** 기준으로 정렬한다.
각 문서는 “스펙(규범) → 구현 → 런북”의 3단 구성을 목표로 한다.

## Phase 0 — Current RFC Baseline (DONE)

- RFC-005: Canon(통합 최종본)
- RFC-001~004: (historical) `rfc/archive/`에 원문 보관

## Phase 1 — Canonical Specs (MUST)

아래 문서가 완성되면 “키 논쟁/정합성 논쟁/Replay 논쟁”이 종결된다.

- `spec/execution-semantics.md`
- `spec/determinism-and-fingerprint.md`
- `spec/authority-and-events.md`
- `spec/raw-and-replay-index.md`
- `spec/admission-and-egress.md`
- `spec/lineage-graph.md`

## Phase 2 — Data Model & APIs (SHOULD)

- `spec/storage-layout.md`
- `spec/api-control-plane.md`
- `spec/api-execution-plane.md`
- `spec/worker-protocol.md`
- `spec/policy-system.md`

## Phase 3 — Operational Runbooks (SHOULD)

- `runbooks/replay-rerun-requeue.md`
- `runbooks/backfill-and-migration.md`
- `runbooks/egress-incident.md`
- `runbooks/schema-drift.md`
- `runbooks/cost-governance.md`

## Phase 4 — Governance & Process (NICE)

- ADR 체계화: `adr/`
- 린트/검증 파이프라인 스펙: `spec/workflow-validation-and-lint.md`
- Studio 운영 가이드: `runbooks/admin-studio.md`

## Definition of Done (for a spec)

각 스펙 문서는 아래를 포함해야 “규범”으로 인정한다.

- Closed-set 정의(상태/열거/필드) 또는 버전화 규칙
- MUST/SHOULD/MAY 규칙(가드/전이/예외)
- 키/다이제스트/정규화 규칙(재현성)
- 최소 구현 체크리스트(테스트 가능)
- 예시 1~2개(가장 위험한 코너케이스 포함)

