# ADR: Idempotency Key Scope (workflow_version 포함 여부)

## Status

- Date: 2026-02-09
- Status: PROPOSED
- Owners: TBD

## Context

RFC들에서 idempotency_key 표기가 `(step_version,input_digest,workflow_version)` 등으로 혼재되어 있다.
하지만 결정론 계약상 결과는 (step_version, canonical_input, task_context, capability_profile)에 의해 결정된다.

## Decision (proposal)

- 기본 idempotency key는 `execution_fingerprint`를 사용한다.
- 기본값으로 `workflow_version`은 fingerprint에 **포함하지 않는다**(강한 결과 재사용).
- 필요 시 워크플로우 단위 캐시 경계를 위해 `cache_scope=workflow_version` 옵션으로 포함할 수 있다.

## Options Considered

1) Always include workflow_version
2) Never include workflow_version (default proposal)
3) Configurable scope (default + override)

## Consequences

- Positive: 캐시 재사용 극대화, 결정론 계약과 정렬
- Negative: 워크플로우 버전 변경이 “의도치 않은 재사용”으로 보일 수 있음(관측/설명 필요)
- Follow-ups:
  - `../spec/determinism-and-fingerprint.md`를 Canon으로 확정
  - UI/운영에서 fingerprint 근거를 설명/노출

## References

- `../spec/determinism-and-fingerprint.md`
- RFC-003

