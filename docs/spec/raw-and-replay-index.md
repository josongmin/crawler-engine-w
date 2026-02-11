# Spec: Raw SSOT, RawIndex, and Replay Lookup

## Status

- Status: DRAFT (seeded from RFC-002/004)

## Purpose

- Raw 불변 원칙을 유지하면서, Replay가 **O(1) 조회**로 결정적으로 동작하도록 한다.

## Raw SSOT (MUST)

- Raw body는 불변(수정/삭제 금지 원칙)
- Raw는 Manifest와 함께 저장되어 provenance를 제공해야 함

## RawIndex (Authority) (MUST)

Replay/Lookup에 필요한 메타데이터 권위 저장소.

### Key

- PK: `source_id#url_digest`
- SK: `fetched_at#attempt_id`

### Attributes (minimum)

- `s3_raw_path`
- `manifest_path`
- `policy_digest`
- `fetcher_profile_digest`
- `status`
- `content_digest`

## Raw Selection Rules (MUST)

Replay는 항상 raw_ref를 **결정적으로** 선택해야 한다.

1. `policy_digest` + `fetcher_profile_digest` 일치 우선
2. 없으면 latest successful
3. 그래도 없으면 Replay 실패

## Compliance Note (TODO)

불변 원칙을 유지하면서도 규정 대응(PII/GDPR/저작권/계약)을 위한 “효과적 삭제” 옵션(키 폐기, 접근 차단, 수명주기)을 별도 스펙으로 분리한다.

## References

- RFC-002
- RFC-004

