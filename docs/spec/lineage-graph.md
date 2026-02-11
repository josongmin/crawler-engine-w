# Spec: Lineage Graph SSOT

## Status

- Status: DRAFT → **NORMATIVE** (seeded from RFC-003/004, SOTA-refined)

## Purpose

- Derived 데이터가 어떤 Raw/Transform chain에서 생성되었는지 **증명 가능**하게 한다.
- Partial recompute / 영향 분석 / 감사(audit)의 기준 SSOT가 된다.
- 학계 SOTA의 Causal Lineage Graph와 정합하는 형식 모델을 제공한다.

## Formal Model (MUST)

Lineage는 **Directed Acyclic Graph (DAG)** 이며:

- **Node** = Artifact (불변, content-addressable)
- **Edge** = Transform (step_version + execution_fingerprint로 식별되는 계산)

### Node Schema (MUST)

| Field | Type | 설명 |
|-------|------|------|
| `artifact_ref` | string | 아티팩트 고유 식별자 (content hash 기반) |
| `artifact_type` | enum | `RAW_ROOT` \| `DERIVED` |
| `created_by_run_id` | string | 생성한 run 식별자 |
| `created_by_task_id` | string | 생성한 task 식별자 |
| `created_at` | ISO 8601 | 생성 시각 |
| `execution_fingerprint` | string | 이 artifact를 생성한 실행의 fingerprint (RAW_ROOT는 null) |

### Edge Schema (MUST)

| Field | Type | 설명 |
|-------|------|------|
| `source_artifact_ref` | string | 입력 artifact |
| `target_artifact_ref` | string | 출력 artifact |
| `step_version` | string | 변환을 수행한 step 버전 |
| `execution_fingerprint` | string | 변환 실행의 fingerprint |
| `run_id` | string | 실행 컨텍스트 |
| `task_id` | string | 태스크 컨텍스트 |

### Minimum Join Key (MUST)

Lineage 조회/조인에 사용되는 최소 키 조합:

```
lineage_join_key = (run_id, task_id, artifact_ref, step_version, execution_fingerprint)
```

이 5-tuple로:
- 특정 artifact의 생성 경로를 역추적할 수 있고
- 특정 실행의 모든 산출물을 열거할 수 있고
- fingerprint 변경 시 영향받는 downstream을 계산할 수 있다

## Normative Rules (MUST)

1. Derived 데이터는 Lineage에 연결되지 않으면 **유효하지 않음**
2. 모든 Artifact는 최소 1개 이상의 upstream 링크(또는 `RAW_ROOT` 표시)를 가져야 함
3. Edge 생성은 `ARTIFACT_COMMITTED` 이후에만 가능 (commit gate 통과 후)
4. Lineage edge는 **불변** (한번 기록되면 수정/삭제 금지)
5. 동일 `execution_fingerprint`로 생성된 artifact는 동일 lineage 구조를 가져야 함

## Partial Recompute (MUST)

### 정의

Partial recompute = 특정 upstream artifact가 변경(또는 무효화)되었을 때, 영향받는 downstream artifact만 선택적으로 재계산하는 연산.

### Algorithm (MUST)

```
partial_recompute_targets(invalidated_artifact_ref):
  1. downstream = BFS(invalidated_artifact_ref, direction=downstream)
  2. for each artifact in downstream:
     a. old_fingerprint = artifact.execution_fingerprint
     b. new_fingerprint = recompute_fingerprint(artifact.inputs)
     c. if old_fingerprint != new_fingerprint:
        → RECOMPUTE_REQUIRED
     d. if old_fingerprint == new_fingerprint:
        → SKIP (idempotency 재사용)
  3. return recompute_required_set
```

### Recompute Scope Rules (MUST)

| 변경 유형 | 영향 범위 | 재계산 방법 |
|----------|----------|-----------|
| Raw 데이터 변경 (새 fetch) | 해당 raw의 모든 downstream | Rerun (새 run_id) |
| step_version 변경 | 해당 step의 모든 downstream | Rerun (새 fingerprint) |
| policy_digest 변경 | capability step의 downstream | Rerun (새 fingerprint) |
| Transform 로직 변경 | 해당 transform의 downstream | Replay 가능 (외부 호출 없음) |

## Query Patterns (MUST)

| Query | 입력 | 출력 | 사용처 |
|-------|------|------|--------|
| Upstream trace | `artifact_ref` | 모든 upstream raw roots | 감사/provenance |
| Downstream impact | `raw_ref` | 영향받는 모든 derived artifacts | 영향 분석 |
| Partial recompute | `invalidated_artifact_ref` | 재계산 필요 artifact set | 부분 재처리 |
| Fingerprint lookup | `execution_fingerprint` | 해당 fingerprint로 생성된 artifacts | 캐시 검증 |
| Run lineage | `run_id` | 해당 run이 생성한 전체 lineage subgraph | 운영 디버깅 |

## Storage (ADR-0003으로 결정)

아래 중 하나를 선택하여 Authority store로 고정한다.

- (Option A) Graph DB — 복잡한 경로 탐색에 유리, 운영 복잡도 높음
- (Option B) RDB adjacency + index — 기존 인프라 재활용, 대규모 DAG에서 성능 한계
- (Option C) Dynamo single-table — query patterns 제한적, fan-out 패턴에 유리

## References

- RFC-003
- RFC-004
- ADR-0003 (Lineage Store Selection)

