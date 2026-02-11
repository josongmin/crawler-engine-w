# Runbook: Replay / Rerun / Requeue

## Purpose

- 재처리 요구(버그 수정, 파서 업그레이드, 일시적 실패, 비용 통제) 시 안전하게 실행한다.

## Preconditions

- 관련 스펙:
  - `../spec/execution-semantics.md`
  - `../spec/raw-and-replay-index.md`
  - `../spec/determinism-and-fingerprint.md`

## Safety Rules (MUST)

- Replay에서는 Capability Step 실행 금지(외부 호출 금지).
- Rerun은 새 `run_id`로 실행(기록 오염 방지).
- Requeue는 동일 Task의 새 Attempt 생성(단, 단일 active attempt 가드 준수).

## Procedure

### 1) Decide mode

- Replay: 외부 호출 없이 Raw/Derived 기반 재계산(빠르고 안전)
- Requeue: 같은 입력으로 같은 Task를 다시 시도(일시 장애)
- Rerun: 전체를 새 run으로 재실행(정의/정책/버전 변경 반영)

### 2) Identify scope

- 대상: `run_id`, `task_key(step_id+shard_key)`, `source_id`, `url_digest`
- Replay는 Raw 선택 규칙이 만족되는지 확인(RawIndex).

### 3) Execute

TODO: Execution Plane API/CLI 기준 절차를 확정 후 구체 명령 작성

### 4) Verify

- Attempt가 VERIFYING/COMMITTED 규칙을 준수했는지 확인
- Idempotency hit 경로(TASK_DEDUPED)로 종료된 태스크 비율 확인

## Postmortem Notes

- replay 실패(원인: raw 없음/정책 mismatch/인덱스 결손) 비율 기록
- 재현성 계약 위반(비결정 요소 누락) 여부 점검

