# ADR: Architecture Decision Map (Big Picture)

## Status

- Date: 2026-02-09
- Status: ACCEPTED (living index)

## Purpose

ADR만 읽어도 “전체 그림”의 **결정 포인트 지형도**를 잡게 한다.
세부 동작/규칙은 `../spec/`가 권위이며, ADR은 “왜/대안/트레이드오프”를 담당한다.

## Canon & Specs (Start Here)

- Canon: `../../rfc/005`
- Specs index: `../spec/README.md`

## Decision Areas → ADRs

### Determinism / Idempotency / Caching

- `0001-idempotency-key-scope.md`

### Authority / Events / Projections

- `0002-outbox-must.md`

### Lineage Store

- `0003-lineage-store-selection.md`

### Compliance / Retention

- `0004-compliance-effective-delete.md`

## “If you change X, review ADRs”

- Idempotency / fingerprint 변경 → ADR-0001
- 외부 워커 dispatch/큐잉 변경 → ADR-0002
- 라인리지 저장소/쿼리 변경 → ADR-0003
- Raw 불변/삭제/접근 정책 변경 → ADR-0004

