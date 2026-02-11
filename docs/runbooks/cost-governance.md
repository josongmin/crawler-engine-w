# Runbook: Cost Governance (Budget, Throttling, Ledger)

## Purpose

- 비용 폭발(요청 수, compute, egress)을 조기에 감지하고, 예산 기반으로 안전하게 throttle/차단/복구한다.

## Preconditions

- 관련 스펙:
  - `../spec/workflow-validation-and-lint.md`
  - `../spec/admission-and-egress.md`
  - `../spec/authority-and-events.md`

## Safety Rules (MUST)

- 예산 초과 시 “정책적으로 실패(RUN_FAILED)” 처리 규칙을 명확히 적용(임의 중단 금지).
- 비용 제어는 admission + scheduler throttling으로 수행(무차별 kill 지양).

## Procedure

### 1) Identify

- 어떤 workflow_version / source_id / domain_group이 비용을 소모 중인가
- 실패율 대비 비용이 비정상적으로 높은 구간이 있는가

### 2) Contain

- max_concurrency 하향
- admission 발급량 제한
- schedule pause

### 3) Remediate

- cost hint 튜닝
- workflow lint 강화(예상 fan-out/예상 비용 임계)

### 4) Recover

- 점진적 복구(예산 범위 내)

## Postmortem Notes

- 비용 폭발 트리거(코드/정의 변경/외부 환경)
- 사전 차단을 위한 lint/시뮬레이션 개선

