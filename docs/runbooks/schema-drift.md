# Runbook: Schema Drift (Validation Fail Spike)

## Purpose

- Validate 실패 급증(드리프트) 발생 시 원인을 격리하고, 새 버전 제안/승인 흐름으로 안전하게 복구한다.

## Preconditions

- 관련 스펙:
  - `../spec/workflow-validation-and-lint.md`
  - `../spec/execution-semantics.md`
  - `../spec/authority-and-events.md`

## Safety Rules (MUST)

- 자동 publish 금지(사람 승인 필요).
- drift 발생 구간은 우선 “확산 방지(ingest throttle/pause)” 후 분석.

## Procedure

### 1) Identify

- 어떤 step_version에서 실패가 급증했는지
- 어떤 source/domain에서 집중되는지
- 실패 유형(필드 누락/타입 변경/HTML 구조 변화/리다이렉트 변화)

### 2) Contain

- 해당 source/domain_group 스케줄 일시 중지 또는 concurrency 하향
- admission 제한(필요 시)

### 3) Remediate

- Admin Studio에서 “새 버전 제안” 생성(스펙상 자동 publish 금지)
- 파서/스키마 변경을 새 step_version으로 publish

### 4) Recover

- 샘플 재실행 → 부분 재처리 → 전체 재개

## Postmortem Notes

- drift_signal 이벤트/프로젝션 정합성 확인
- lint/validation 규칙 보강 포인트

