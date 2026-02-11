# Spec: Execution Semantics (Run/Task/Attempt/Artifact)

## Status

- Status: DRAFT → **NORMATIVE** (seeded from RFC-001, SOTA-refined)
- Depends on: `determinism-and-fingerprint.md` (idempotency key, step_type)

## Purpose

- 운영 지옥 케이스(중복 실행, 외부 워커 지연, out-of-order 이벤트, lease 만료, 커밋 실패)에서 **해석 여지 0**의 실행 의미론 제공.

## Source of Truth

- Authority: Execution Authority = **DynamoDB** (단일 권위자)
- Projection: RDS 요약/리포트는 projection이며 SSOT 아님.

## Closed Sets (Normative)

이 문서에서는 상태 열거형을 **Closed Set**으로 정의하며 **임의 확장 금지**.

### RunState

| State | Terminal | 설명 |
|-------|----------|------|
| `RUN_PENDING` | N | 생성됨, Task 생성 전 |
| `RUN_RUNNING` | N | 1개 이상의 Task가 활성 |
| `RUN_PAUSED` | N | 운영자/정책에 의해 일시정지 |
| `RUN_SUCCEEDED` | **Y** | 모든 Task가 성공 또는 deduped |
| `RUN_FAILED` | **Y** | 1개 이상 Task가 최종 실패 |
| `RUN_CANCELLED` | **Y** | 운영자에 의해 취소 |

### TaskState

| State | Terminal | 설명 |
|-------|----------|------|
| `TASK_PENDING` | N | 생성됨, 의존성 대기 |
| `TASK_READY` | N | 의존성 충족, idempotency 조회 직전 |
| `TASK_DEDUPED` | **Y** | idempotency hit → Attempt 생성 없이 artifact_ref 연결 |
| `TASK_RUNNING` | N | active Attempt 존재 |
| `TASK_SUCCEEDED` | **Y** | Attempt가 SUCCEEDED |
| `TASK_FAILED` | **Y** | 최종 실패 (retry 소진 또는 정책 판정) |

### AttemptState

| State | Terminal | 설명 |
|-------|----------|------|
| `ATTEMPT_LEASED` | N | lease 획득, 워커에 dispatch 직전/직후 |
| `ATTEMPT_RUNNING` | N | 워커가 실행 중 (heartbeat 활성) |
| `ATTEMPT_VERIFYING` | N | 산출물 검증 진행 중 |
| `ATTEMPT_SUCCEEDED` | **Y** | Artifact COMMITTED 확인 후 성공 |
| `ATTEMPT_FAILED` | **Y** | 실행 실패 또는 검증 실패 |
| `ATTEMPT_LEASE_EXPIRED` | **Y** | lease 만료 (heartbeat 부재) |

### ArtifactState

| State | Terminal | 설명 |
|-------|----------|------|
| `ARTIFACT_STAGED` | N | presigned upload 완료, 검증 대기 |
| `ARTIFACT_COMMITTED` | **Y** | 검증 통과, 불변 확정 |
| `ARTIFACT_REJECTED` | **Y** | 검증 실패 |

## Invariants (MUST)

1. **단조성**: 이전 상태로 되돌림 금지. 모든 전이는 forward-only
2. **단일 활성 Attempt**: Task당 동시에 1개의 active Attempt만 허용. Active = `{LEASED, RUNNING, VERIFYING}`
3. **Commit Gate**: Attempt 성공은 Artifact `COMMITTED` 이후에만 가능. `VERIFYING` 통과 필수
4. **Replay 차단**: Replay 모드에서 `Capability` step의 Attempt 생성 금지 (dispatch 레벨 거부)
5. **Idempotency 선행**: `TASK_READY` 진입 시 반드시 idempotency 조회 선행. hit이면 `TASK_DEDUPED` (Attempt 미생성)

## State Transition Table (MUST)

### Run Transitions

| From | To | Guard (conditional write) |
|------|----|--------------------------|
| `RUN_PENDING` | `RUN_RUNNING` | 최초 Task dispatch 시 |
| `RUN_RUNNING` | `RUN_SUCCEEDED` | 모든 Task ∈ {SUCCEEDED, DEDUPED} |
| `RUN_RUNNING` | `RUN_FAILED` | 1+ Task FAILED ∧ retry 소진 |
| `RUN_RUNNING` | `RUN_PAUSED` | 운영자/정책 pause 요청 |
| `RUN_PAUSED` | `RUN_RUNNING` | resume 요청 |
| `RUN_RUNNING` | `RUN_CANCELLED` | cancel 요청 |
| `RUN_PAUSED` | `RUN_CANCELLED` | cancel 요청 |

### Task Transitions

| From | To | Guard |
|------|----|-------|
| `TASK_PENDING` | `TASK_READY` | 의존 Task 전부 ∈ {SUCCEEDED, DEDUPED} |
| `TASK_READY` | `TASK_DEDUPED` | idempotency hit |
| `TASK_READY` | `TASK_RUNNING` | idempotency miss → Attempt 생성 |
| `TASK_RUNNING` | `TASK_SUCCEEDED` | Attempt SUCCEEDED |
| `TASK_RUNNING` | `TASK_READY` | Attempt 실패 ∧ retry 가능 → 새 Attempt 대기 |
| `TASK_RUNNING` | `TASK_FAILED` | Attempt 실패 ∧ retry 소진 |

### Attempt Transitions

| From | To | Guard |
|------|----|-------|
| (생성) | `ATTEMPT_LEASED` | conditional write: Task에 active attempt 없음 |
| `ATTEMPT_LEASED` | `ATTEMPT_RUNNING` | 워커 첫 heartbeat 수신 |
| `ATTEMPT_RUNNING` | `ATTEMPT_VERIFYING` | 워커 commit_artifact 호출 |
| `ATTEMPT_VERIFYING` | `ATTEMPT_SUCCEEDED` | Artifact COMMITTED |
| `ATTEMPT_VERIFYING` | `ATTEMPT_FAILED` | Artifact REJECTED |
| `ATTEMPT_LEASED` | `ATTEMPT_LEASE_EXPIRED` | heartbeat 부재 + lease TTL 초과 |
| `ATTEMPT_RUNNING` | `ATTEMPT_LEASE_EXPIRED` | heartbeat 부재 + lease TTL 초과 |
| `ATTEMPT_RUNNING` | `ATTEMPT_FAILED` | 워커 error 보고 |

### Artifact Transitions

| From | To | Guard |
|------|----|-------|
| (생성) | `ARTIFACT_STAGED` | 서버가 S3 head 확인 성공 |
| `ARTIFACT_STAGED` | `ARTIFACT_COMMITTED` | hash + size + manifest + schema 검증 통과 |
| `ARTIFACT_STAGED` | `ARTIFACT_REJECTED` | 검증 실패 |

## Transition Rules (MUST)

### Conditional Write Only

모든 전이는 DynamoDB conditional write로만 발생해야 함. 낙관적 잠금이 아니라 **조건부 갱신(condition expression)** 으로 원자적 보장.

### Lease Model (MUST)

**권위자**: Execution Authority(DynamoDB)가 lease의 **단일 권위자**.

| 항목 | 규칙 |
|------|------|
| Lease 생성 | `ATTEMPT_LEASED` 전이 시 `lease_expiry = now() + lease_duration` 기록 |
| Lease Duration | 기본 300초, policy_override로 step별 조정 가능 |
| Heartbeat | 워커는 `lease_duration / 3` 간격으로 heartbeat 전송 |
| Heartbeat 수신 | `lease_expiry = now() + lease_duration` 갱신 (conditional write: 현재 attempt가 active인 경우만) |
| Clock Skew 허용 | 서버 시간 기준. 워커/서버 간 NTP 동기화 전제. skew 허용 범위 = **5초** |
| Lease 만료 판정 | Reaper 프로세스가 `lease_expiry < now() - skew_tolerance` 인 attempt를 `ATTEMPT_LEASE_EXPIRED`로 전이 |
| 만료 후 워커 보고 | conditional write 실패 → 워커 보고 무시 (무해화) |

### Dispatching (External Workers) (MUST)

- enqueue(SQS) + 상태전이(Dynamo)는 **Dynamo outbox로 원자성 확보** (ADR-0002 MUST 채택)
- Outbox processor가 outbox 레코드를 읽어 SQS enqueue → 완료 시 outbox 마킹
- 중복 enqueue는 워커의 idempotent 처리 + conditional write로 무해화

## Cancellation / Pause Semantics (MUST)

### RUN_PAUSED

| 대상 | 영향 |
|------|------|
| 새 Attempt 생성 | **금지** (TASK_READY → TASK_RUNNING 전이 차단) |
| 기존 ATTEMPT_RUNNING | **graceful stop 요청** 발행. lease 만료 시 자동 EXPIRED |
| 기존 ATTEMPT_LEASED | lease 갱신 금지 → 자연 만료 |

### RUN_CANCELLED

| 대상 | 영향 |
|------|------|
| 미시작 Task | TASK_PENDING → TASK_FAILED (cancel reason) |
| 활성 Attempt | graceful stop + lease 갱신 금지. 만료 시 EXPIRED |
| 이미 완료 Task | 상태 유지 (cancel은 forward-only 원칙을 위반하지 않음) |

## Requeue Semantics (MUST)

1. 동일 Task에 새 Attempt 생성 (conditional write: active attempt 없음 확인)
2. `input_digest`는 **원본 유지** (동일 fingerprint)
3. retry 정책에 따라 backoff/격리 적용
4. 성공 시 `idempotency_record` upsert (동일 fingerprint)

## Minimal Implementation Checklist

- [ ] 상태/전이 전부 conditional write
- [ ] Attempt VERIFYING 단계 강제
- [ ] idempotency hit 경로는 TASK_DEDUPED로 terminal 처리 (Attempt 미생성)
- [ ] Replay에서 Capability step dispatch 거부
- [ ] Lease reaper 프로세스 구현
- [ ] Outbox processor 구현 (ADR-0002)
- [ ] Pause/Cancel 시 active attempt graceful stop

## References

- RFC-001
- RFC-004
- ADR-0002 (Outbox MUST)

