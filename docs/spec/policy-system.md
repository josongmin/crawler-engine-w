# Spec: Source Policy System

## Status

- Status: DRAFT
- Depends on: `raw-and-replay-index.md`, `admission-and-egress.md`

## Purpose

- 도메인/소스별 정책을 “preset + override”로 관리하고, 모든 실행이 `policy_digest`로 재현 가능하도록 한다.

## Policy Digest (MUST)

- 정책 변경은 digest 변경으로만 반영되어야 한다.
- 과거 실행 재현을 위해 (workflow_version, policy_digest) 조합을 기록한다.

## Model

### Preset

- 도메인군 정책 템플릿
- 예: robots 준수, crawl-delay, rate limit, user-agent class, rendering, retries

### Override

- 소스/도메인별 최소 필드만 덮어쓰기
- 전체 정책은 preset + override merge 결과

## Required Linkage (MUST)

- Raw manifest에 `policy_digest`를 기록
- RawIndex에 `policy_digest`를 인덱싱

## Open Questions (ADR candidates)

- robots 판단/캐시/만료 규칙
- “politeness”와 admission(egress 토큰) 결합 방식

## References

- RFC-002
- RFC-004

