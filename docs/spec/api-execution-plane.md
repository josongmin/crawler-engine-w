# Spec: Execution Plane API

## Status

- Status: DRAFT
- Authority: DynamoDB (Execution Authority)

## Purpose

- Run/Task/Attempt lifecycle 제어 및 조회 API 계약을 고정한다.

## Normative Rules (MUST)

- 상태 전이는 모두 conditional write 기반(서버가 가드 수행)
- Exactly-once 실행 보장 주장 금지(At-least-once + idempotency 결과 재사용)
- 외부 워커 산출물은 COMMITTED 전 성공 불가

## Resources (minimum)

- Run
- Task
- TaskAttempt
- Artifact
- IdempotencyRecord

## Endpoints (skeleton)

### Runs

- `POST /runs` (start run; input: workflow_version, params, mode)
- `GET /runs/{run_id}`
- `POST /runs/{run_id}/cancel`
- `POST /runs/{run_id}/pause`
- `POST /runs/{run_id}/resume`

### Tasks

- `GET /runs/{run_id}/tasks`
- `POST /tasks/{task_key}/requeue`

### Replay / Rerun

- `POST /runs/{run_id}/replay` (capability steps disabled)
- `POST /runs/{run_id}/rerun` (new run_id)

### Artifacts

- `GET /artifacts/{artifact_ref}`

## Query Requirements (MUST)

- run_id 기준 E2E 조회
- step_id/step_version 기준 집계
- source_id/url_digest 기준 추적(Replay/RawIndex 연동)

## References

- RFC-001
- RFC-004

