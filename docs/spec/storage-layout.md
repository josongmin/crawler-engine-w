# Spec: Storage Layout (RDS/DynamoDB/S3)

## Status

- Status: DRAFT
- Depends on: `authority-and-events.md`, `execution-semantics.md`, `raw-and-replay-index.md`

## Purpose

- 각 저장소의 **권위(Authority) 범위**와 테이블/버킷 레이아웃을 고정한다.
- 쿼리 패턴/인덱스/핫파티션 리스크를 문서 레벨에서 선제 차단한다.

## Authority Summary (MUST)

- RDS: Definition Authority
- DynamoDB: Execution Authority, Egress Authority
- S3: Raw/Derived immutable blobs (권위 메타는 index store가 담당)

## RDS (Definition Authority)

### Tables (candidate)

- `workflows`
- `workflow_versions`
- `schedules`
- `sources`
- `step_catalog`
- `policy_presets`

### Required properties (minimum)

TODO: 스키마 초안(키/버전/불변 규칙 포함)

## DynamoDB (Execution Authority)

### Tables (candidate)

- `runs`
- `tasks`
- `task_attempts`
- `idempotency_records`
- `locks`
- `raw_index` (RawIndex 권위가 Dynamo인 경우)
- `egress_state`

### Required patterns (MUST)

- Run 단위 조회
- Task fan-out 조회(파티션/정렬키 설계)
- Attempt 최신 상태 조회
- Idempotency O(1) 조회

TODO: 단일테이블 vs 멀티테이블 ADR 필요

## S3 (Raw/Derived)

### Buckets / Prefixes

- `raw/`
- `derived/`
- `logs/`
- `debug/`

### Manifest (MUST)

Raw object는 항상 manifest와 함께 저장되어야 한다.

## References

- RFC-002
- RFC-004

