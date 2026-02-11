# Spec: Compliance & Retention (Effective Delete)

## Status

- Status: DRAFT
- Depends on: `raw-and-replay-index.md`

## Purpose

- Raw 불변 원칙을 유지하면서도 규정(GDPR/PII/저작권/계약) 대응을 가능하게 한다.

## Data Classification (TODO)

- 등급 정의: PII / Restricted / Public
- 등급별 저장/접근/로그 정책

## Effective Delete (MUST)

“Hard delete(물리 삭제)”가 아니라 “접근 불가능화”를 우선한다.

- Key revoke/rotate 기반 접근 불가화(KMS)
- ACL/Policy 기반 접근 차단
- Index 숨김 및 감사 로그 기록

## Retention Policies (TODO)

- raw/derived/index/lineage 각각 보관기간/아카이브 규칙
- legal hold 규칙

## References

- ADR-0004
- RFC-002
- RFC-004

