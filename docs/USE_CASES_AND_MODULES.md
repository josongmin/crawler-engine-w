# Use Cases & Module Map (One-page)

목적: 개발 착수 전에 "무엇을 어떤 컴포넌트가 책임지는지"를 한 장으로 고정한다.
최종 규범은 `docs/spec/`이며, Canon은 `rfc/005`.

## 0. SOTA 아키텍처 원칙 (Anchor Points)

> 아래 8개 원칙은 모든 UC/모듈에 선행하며, 구현 시 해석 여지를 없앤다.

| # | 원칙 | 한 줄 고정 |
|---|------|-----------|
| P1 | Dual Authority 분리 | Definition Authority(RDS)와 Execution Authority(Dynamo)는 교차 트랜잭션 금지. 교차 데이터는 Projection일 뿐 SSOT 아님 |
| P2 | Immutable Versioning | workflow_version, artifact, raw blob은 불변. 수정은 항상 새 버전/새 run_id로만 표현 |
| P3 | Deterministic Fingerprinting | `execution_fingerprint = hash(step_version + input_digest + canonical_spec_version + determinism_context_digest + capability_profile_digest)`. 캐시/중복제거/재현성의 유일 기준 |
| P4 | Idempotency as First-class | 모든 Task는 실행 전 idempotency 조회가 선행. "실행"이 아니라 "결과 확정"이 비용 단위 |
| P5 | Commit Gate Pattern | staged → verify → committed 3단계. 성공 정의는 "실행 완료"가 아니라 "검증 통과 후 commit" |
| P6 | Event-sourced Projection | 상태는 authority에, 읽기 모델은 projection에. projector는 역순·중복 이벤트에도 무해 |
| P7 | Capability Isolation | 외부 I/O step은 capability step으로 명시. replay 시 capability는 구조적으로 실행 불가 |
| P8 | Lineage-complete System | 모든 artifact는 upstream lineage로 역추적 가능. partial recompute가 자동 계산 가능해야 함 |

### 빠진 고정점 6개 (착수 전 반드시 닫아야 하는 항목)

| # | 고정점 | 고정 문장 | 규범 위치 |
|---|--------|----------|----------|
| F1 | IR/DSL vs Spec/Canon 관계 | WorkflowVersion의 저장 포맷/스키마 SSOT는 **Workflow IR**이다. DSL은 Studio 내부 편집 포맷이며, DAG는 IR의 View(Projection)이다 | `rfc/005` Law-SSOT-IR-1 |
| F2 | Fingerprint 포함/제외 항목 | MUST 포함: `step_version`, `input_digest`, `canonical_spec_version`, `determinism_context_digest`, `capability_profile_digest`. `workflow_version`은 기본 제외(ADR-0001) | `spec/determinism-and-fingerprint.md` |
| F3 | Lease 권위자와 규칙 | Lease는 **Execution Authority(Dynamo)**가 단일 권위자. renew/expire는 Attempt 상태 전이와 묶이며, lease_expiry 초과 시 ATTEMPT_LEASE_EXPIRED → 새 Attempt 가능 | `spec/execution-semantics.md` |
| F4 | Artifact Commit 검증 스펙 | Commit 검증은 hash 무결성 + size 일치 + manifest 존재 + schema 적합성(해당 시)을 통과해야 COMMITTED. 하나라도 실패 시 ARTIFACT_REJECTED | `spec/worker-protocol.md` |
| F5 | Outbox 원자성 경계 | Definition publish는 **RDS outbox**, execution dispatch는 **Dynamo outbox**로 경계를 분리한다. 각 authority 내에서만 원자성 보장 | `spec/authority-and-events.md` + ADR-0002 |
| F6 | Lineage 최소 조인 키 | Lineage edge는 `(run_id, task_id, artifact_ref, step_version, execution_fingerprint)` 5-tuple로 조인된다 | `spec/lineage-graph.md` |

## A. 주요 유즈케이스별 흐름 (E2E)

### UC1) Workflow Version Publish (정의 배포)

1) Studio/API에서 draft IR/DSL 생성  
2) 정적 검증 + lint 통과 (`spec/workflow-validation-and-lint.md`)  
3) RDS 트랜잭션 내 publish 확정 (`spec/api-control-plane.md`)  
4) `workflow_version_published` 이벤트 발행 (`spec/authority-and-events.md`)  

### UC2) Scheduled Run Start → Fan-out 실행

1) Scheduler가 (workflow_version, params)로 Run 생성 요청  
2) Execution Authority(Dynamo)에 `Run/Task/Attempt` 생성 및 상태 전이 (`spec/execution-semantics.md`)  
3) Fan-out은 큐(SQS) + 워커풀로 분리, backpressure 적용 (`rfc/005`)  
4) Task가 `TASK_READY` 진입 시 idempotency 조회 → hit면 `TASK_DEDUPED` (`spec/determinism-and-fingerprint.md`)  
5) miss면 Attempt leased/dispatched → running

### UC3) Capability Step 실행 + Artifact Commit Gate

1) Admission Control로 `admission_token` 획득 (`spec/admission-and-egress.md`)  
2) 워커 실행(내부/외부/디바이스) (`spec/worker-protocol.md`)  
3) 산출물은 presigned 업로드 → `ARTIFACT_STAGED`  
4) 서버 검증 후 `ARTIFACT_COMMITTED`  
5) Attempt는 VERIFYING 통과 후에만 `ATTEMPT_SUCCEEDED` 가능 (`spec/execution-semantics.md`)

### UC4) Replay (외부 호출 금지 재처리)

1) Replay 모드로 실행 요청  
2) RawRef는 RawIndex 규칙으로 결정 (`spec/raw-and-replay-index.md`)  
3) Capability step은 실행 금지, Transform/Pure만 수행 (`spec/determinism-and-fingerprint.md`)  

### UC5) Requeue (특정 Task 재시도)

1) 동일 Task에 새 Attempt 생성(단일 active attempt 가드)  
2) input_digest 유지, 정책에 따라 retry/격리  
3) 성공 시 idempotency_record upsert  

### UC6) Rerun (전체 새 run_id 재실행)

1) new run_id 생성  
2) idempotency hit로 결과 재사용 가능(캐시 경계는 ADR-0001)  

### UC7) Drift Signal 대응(스키마/HTML 변화)

1) Validate 실패 임계치 초과 → drift_signal 이벤트  
2) 자동 publish 금지, Studio가 새 버전 제안 생성 + 승인  
3) 샘플 → 부분 재처리 → 전체 재개 (`runbooks/schema-drift.md`)

### UC8) Egress Incident (ban storm / capacity drop)

1) domain_group/proxy_pool_group 범위 식별  
2) admission 발급 제한 + schedule throttle/pause  
3) cooldown/대체 pool 적용 후 점진 복구 (`runbooks/egress-incident.md`)

## B. 공통 모듈 맵 (반복 등장 컴포넌트)

### 1) Control Plane (Definition Authority)

- **Workflow Service**: workflow/workflow_version CRUD, publish(불변)  
- **Validation/Lint Service**: publish gate(정적 검증/리스크 차단)  
- **Policy Service**: preset+override → `policy_digest` 산출 (`spec/policy-system.md`)  
- **Studio**: IR 편집, preview, diff/rollback, drift 제안

Storage/SSOT:

- RDS(Definition Authority)

### 2) Execution Plane (Execution Authority)

- **Run/Task/Attempt State Machine**: conditional write 전이, invariants 강제  
- **Idempotency Store**: `idempotency_key(execution_fingerprint)` → artifact_ref  
- **Artifact Commit Service**: staged→verify→commit gate  
- **Dispatcher/Leaser**: 내부 lease 또는 외부 dispatch(outbox 연동)

Storage/SSOT:

- DynamoDB(Execution Authority)

### 3) Worker Runtime (Internal / External / Device)

- **Worker Agent**: task 실행, 업로드, 결과 보고  
- **Capability Profiles**: network/egress/proxy/device 프로필 표준화  

### 4) Raw & Replay

- **Raw Store**: S3 immutable blobs + manifest  
- **RawIndex**: replay lookup authority (`source_id#url_digest` 인덱스)  

### 5) Admission & Egress (Physical Backpressure)

- **Admission Service**: 토큰 발급/반납/만료(정책 TBD)  
- **Egress SSOT**: ProxyPoolGroup/IPState/BanHistory/Cooldown  

### 6) Events & Projections (Cross-domain glue)

- **Domain Event Envelope**: event_id/aggregate_version 포함  
- **Projectors**: idempotent projection (역순/중복 무해화)  
- **Outbox**: dispatch/enqueue 원자성(ADR-0002)

### 7) Determinism Library (공통 라이브러리)

- **Canonicalization** + `input_digest`  
- **Fingerprint**: `execution_fingerprint` 계산  
- **TaskContext normalization** (seed/locale 등)

### 8) Lineage

- **Lineage Writer**: artifact 노드/edge 기록  
- **Query API**: upstream/downstream, partial recompute 대상 계산  

## C. "새 기능 추가 시 어디를 만지나" (빠른 라우팅)

- 새 fetcher/디바이스 추가: Worker Protocol + Capability Profile + Admission/Egress
- 재현성/캐시 정책 변경: Determinism/Fingerprint + ADR-0001
- 큐/디스패치 경로 변경: Execution Semantics + Authority/Events + ADR-0002(outbox)
- Replay 성능/규칙 변경: RawIndex/Selection rules
- 부분 재계산/영향 분석: Lineage store/query patterns

## D. 설계 결정 레지스터 (촌철살인 질문 → 고정 응답)

> 착수 전에 해석이 갈리는 8개 질문에 대해 설계 결정을 고정한다.

| # | 질문 | 고정 응답 | 규범 위치 |
|---|------|----------|----------|
| Q1 | Idempotency hit 시 Attempt를 만드는가? | **만들지 않는다.** `TASK_READY` → idempotency 조회 → hit이면 곧바로 `TASK_DEDUPED`(terminal). Attempt 생성 없이 artifact_ref를 연결한다 | `spec/execution-semantics.md` |
| Q2 | ARTIFACT_STAGED는 누가 기록하는가? | **서버가 기록한다.** Worker가 presigned upload 완료 후 `commit_artifact(artifact_digest, meta)` 호출 → 서버가 S3 head 확인 → STAGED 기록 → 검증 시작 | `spec/worker-protocol.md` |
| Q3 | Replay에서 Pure/Transform 판정 기준은? | **Workflow IR의 step_type 태깅.** 각 step은 `Pure`, `Transform`, `Capability` 중 하나로 선언되며, Replay 모드는 `Capability` step의 Attempt 생성을 구조적으로 차단한다 | `spec/determinism-and-fingerprint.md` |
| Q4 | "단일 active attempt 가드"의 정의는? | **ATTEMPT_LEASED ∪ ATTEMPT_RUNNING ∪ ATTEMPT_VERIFYING** 중 하나라도 존재하면 새 Attempt 생성 금지. ATTEMPT_FAILED/EXPIRED만 남은 경우에만 새 Attempt 가능 | `spec/execution-semantics.md` |
| Q5 | Rerun에서 policy_digest 변경 시 idempotency 재사용? | **재사용 불가.** `policy_digest`가 `capability_profile_digest`에 영향 → `execution_fingerprint`가 달라짐 → idempotency miss → 새 실행. 이것이 ADR-0001의 "강한 결과 재사용" 설계 의도 | ADR-0001 + `spec/determinism-and-fingerprint.md` |
| Q6 | Drift signal 임계치 측정 단위는? | **workflow_version × source_group 단위.** 동일 workflow_version에서 특정 source_group의 validate 실패율이 임계치(기본 30%, 정책으로 override 가능) 초과 시 drift_signal 발행 | `spec/policy-system.md` + `runbooks/schema-drift.md` |
| Q7 | Admission token 발급 단위는? | **Attempt 단위.** 하나의 Attempt에 하나의 admission_token. token은 attempt 종료(성공/실패/만료) 시 자동 반납. 미반납 시 TTL(lease_duration 연동) 후 만료 회수 | `spec/admission-and-egress.md` |
| Q8 | Outbox 중복 소비 시 무해화 키는? | **`(aggregate_id, aggregate_version)` 쌍.** Projector는 이미 처리한 version 이하는 skip. Dispatcher는 `event_id`로 중복 enqueue 방지(idempotent put) | `spec/authority-and-events.md` |

