# Runbook: Backfill & Migration

## Purpose

- 스키마/파서/정책 변경 후 과거 데이터를 안전하게 백필(재처리)한다.
- 대규모 재처리 시 비용/egress 폭발을 방지한다.

## Preconditions

- 관련 스펙:
  - `../spec/determinism-and-fingerprint.md`
  - `../spec/lineage-graph.md`
  - `../spec/admission-and-egress.md`
  - `../spec/workflow-validation-and-lint.md`

## Safety Rules (MUST)

- 백필은 단계적(샘플 → 부분 → 전체)으로 진행
- Admission/예산(budget)로 물리 자원 한계 강제
- Derived는 반드시 lineage 연결(없으면 무효)

## Procedure

### 1) Plan

- 변경 사항: step_version / parser_version / policy_digest / canonical_spec_version
- 백필 범위: source_id, 기간, url_digest set

### 2) Dry-run

- 소량 샘플로 replay 가능성(raw 존재 여부) 검증
- cost hint 기반 예상 비용 산출

### 3) Execute

TODO: partial recompute(라인리지 기반) 우선 백필 절차 구체화

### 4) Validate

- 성공률/비용/egress/ban_rate
- drift signal 여부

## Postmortem Notes

- 라인리지 결손 원인
- raw 인덱스 결손 원인

