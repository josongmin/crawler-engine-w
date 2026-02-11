# Spec: Workflow Validation & Lint

## Status

- Status: DRAFT
- Depends on: `determinism-and-fingerprint.md`, `admission-and-egress.md`

## Purpose

- Publish 전에 구조적 결함(비용 폭발, 무한 fan-out, sink 없음, 과도 retry)을 차단한다.
- IR/DSL이 장기 호환되도록 “정적 검증 가능한 형태”를 강제한다.

## Validation (MUST)

- 순환 참조 금지
- input/output schema mismatch 금지
- fan-out 상한(글로벌/step별) 강제
- 글로벌 timeout/time_budget 강제
- capability step은 admission_required 설정 규칙 준수
- canonical_spec_version 필수

## Lint Rules (MUST/SHOULD)

### MUST (publish block)

- 무한 fan-out 패턴
- sink 없는 흐름
- retry 폭발(상한 초과)
- budget envelope 누락

### SHOULD (warning)

- hot domain 집중 위험(egress profile 불균형)
- 예상 비용 임계 초과

## Studio Integration

- Predictive simulation: 예상 fan-out/비용/실패 hotspot 표시
- Risk score: 임계 초과 시 publish 차단

## References

- RFC-002
- RFC-003

