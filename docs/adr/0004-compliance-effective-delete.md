# ADR: Compliance “Effective Delete” (불변 원칙 유지)

## Status

- Date: 2026-02-09
- Status: PROPOSED
- Owners: TBD

## Context

Raw 불변(삭제/수정 금지) 원칙은 재현성과 감사에 매우 유리하지만,
PII/GDPR/저작권/계약 등 규정 이슈가 현실적으로 발생한다.

## Decision (proposal)

불변 원칙을 유지하면서도 규정을 만족하기 위해 “효과적 삭제” 전략을 지원한다.

- (Option A) 객체 암호화 키 폐기(KMS key revoke/rotate)로 실질 접근 불가화
- (Option B) 접근 제어 정책으로 차단 + 인덱스에서 숨김(원본은 보관)
- (Option C) 법적 허용 범위 내 수명주기 정책(보관등급/만료)

## Options Considered

1) Hard delete (원칙 위반 → 비권장)
2) Effective delete (proposal)
3) Hybrid (데이터 등급별)

## Consequences

- Positive: 재현성 철학 유지 + 규정 대응 가능
- Negative: 운영 복잡도 증가(키/ACL/인덱스 동기화)
- Follow-ups:
  - compliance 스펙 분리 문서 생성: `../spec/compliance-and-retention.md`

## References

- `../spec/raw-and-replay-index.md`
- RFC-002
- RFC-004

