# Runbook: Admin Studio 운영 가이드

## Purpose

- DAG/IR Studio를 운영하면서 publish, rollback, preview, drift 대응을 안전하게 수행한다.

## Preconditions

- 관련 스펙:
  - `../spec/workflow-validation-and-lint.md`
  - `../spec/authority-and-events.md`
  - `../spec/determinism-and-fingerprint.md`

## Safety Rules (MUST)

- Publish는 항상 “새 workflow_version 생성”으로만 수행(기존 버전 수정 금지).
- 자동 publish 금지(사람 승인 필요).
- 위험 lint(무한 fan-out, sink 없음, 과도 retry, 예산 누락)는 publish 차단.

## Procedure (skeleton)

### 1) Design / Edit

- IR 편집(또는 DSL 편집) 원칙 확인
- Subgraph로 큰 DAG 분할

### 2) Preview Run

- 샘플 입력으로 preview
- 예상 fan-out/비용/실패 hotspot 확인

### 3) Publish

- 정적 검증 + lint 통과 확인
- publish 이벤트 발행 확인

### 4) Rollback

- 이전 버전 재활성화 방식(새 버전 생성 없이 “활성 포인터”만 변경 여부는 정책에 따름)

### 5) Drift 대응

- drift signal 수신 → 새 버전 제안 생성 → 승인 → 부분 재처리/재개

