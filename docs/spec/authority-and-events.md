# Spec: Authority Split, Domain Events, and Projections

## Status

- Status: DRAFT → **NORMATIVE** (seeded from RFC-004, SOTA-refined)

## Purpose

- "RDS vs Dynamo 정합성" 같은 논쟁을 종결한다.
- 교차 트랜잭션을 금지하고, **Authority + Event + Projection**으로만 도메인 경계를 연결한다.

## Authority (MUST)

각 도메인은 단 하나의 권위 저장소(Authority store)를 가진다. **교차 트랜잭션 금지.**

| Domain | Authority Store | 책임 범위 |
|--------|----------------|----------|
| Definition | **RDS** | workflow, workflow_version, schedule, source, step_catalog, policy_preset |
| Execution | **DynamoDB** | run, task, attempt, idempotency_record, lock |
| Raw | **RawIndex(DynamoDB)** + S3(immutable blob) | raw 메타/선택 규칙(RawIndex), raw body(S3) |
| Lineage | **Lineage Store** (ADR-0003으로 선정) | artifact node, transform edge |
| Egress | **DynamoDB** | ProxyPoolGroup, IPState, BanHistory, CooldownWindow |

> **교차 데이터 원칙**: Authority 간 데이터 참조는 **Projection**일 뿐이며 SSOT가 아니다. Projection은 eventual consistency를 전제한다.

## Domain Events (MUST)

### Event Envelope

모든 도메인 이벤트는 최소한 아래 필드를 가진다.

| Field | Type | 설명 |
|-------|------|------|
| `event_id` | ULID | 이벤트 고유 식별자. 중복 감지 키 |
| `event_type` | string | 이벤트 타입 (예: `workflow_version_published`) |
| `occurred_at` | ISO 8601 | 이벤트 발생 시각 |
| `producer` | string | 발행 서비스 식별자 |
| `aggregate_type` | string | 집합 타입 (예: `workflow_version`, `run`, `attempt`) |
| `aggregate_id` | string | 집합 인스턴스 식별자 |
| `aggregate_version` | int64 | 단조 증가 버전 (optimistic concurrency) |
| `payload` | JSON | 이벤트 본문 |

### Idempotent Projection Rule (MUST)

Projector는 **두 가지 키**로 중복/역순 이벤트를 무해화해야 한다.

1. **Projector 내 무해화**: `(aggregate_id, aggregate_version)` 기준
   - 이미 처리한 version >= incoming version → **skip**
   - gap 발생 시 → 재시도 큐로 전송, gap 해소 후 재처리
2. **Dispatcher 내 중복 방지**: `event_id` 기준
   - 동일 `event_id`로 중복 enqueue 시 → **idempotent put** (skip)

### Gap Handling Rule (MUST)

- Projector는 `last_processed_version`을 유지
- incoming `aggregate_version` > `last_processed_version + 1` → gap 감지
- gap 이벤트는 **dead-letter가 아닌 retry queue**로 전송
- gap 해소 대기 시간 초과(기본 60초) → alert + manual intervention

## Outbox Pattern (MUST) — ADR-0002 채택

큐 enqueue / 상태 갱신처럼 "부수효과 + 상태전이"가 함께 필요한 구간은 outbox로 원자성을 확보한다.

### Outbox 경계 분리 (MUST)

| Authority | Outbox 위치 | 원자성 범위 | 소비자 |
|-----------|------------|-----------|--------|
| **Definition (RDS)** | RDS outbox 테이블 | RDS 트랜잭션 내에서 비즈니스 레코드 + outbox 레코드 동시 insert | Event bus (SNS/EventBridge) |
| **Execution (DynamoDB)** | DynamoDB outbox 항목 | DynamoDB TransactWriteItems로 상태전이 + outbox 항목 동시 write | SQS enqueue processor |

### Outbox Processor Rules (MUST)

1. Outbox processor는 **polling 또는 DynamoDB Streams**로 outbox 레코드를 소비
2. 소비 후 대상 시스템(SQS/SNS)에 전달 → 성공 시 outbox 마킹(processed)
3. 중복 전달은 **소비자 측 idempotency**로 무해화 (`event_id` 기준)
4. Outbox processor 장애 시 미처리 레코드는 재처리 (at-least-once)

### Domain Events Catalog (Normative)

| Event Type | Producer | Authority | 핵심 Payload |
|-----------|----------|-----------|-------------|
| `workflow_version_published` | Control Plane | Definition(RDS) | workflow_version_id, policy_digest |
| `run_created` | Execution | Execution(Dynamo) | run_id, workflow_version_id |
| `task_state_changed` | Execution | Execution(Dynamo) | task_id, from_state, to_state |
| `attempt_state_changed` | Execution | Execution(Dynamo) | attempt_id, from_state, to_state |
| `artifact_committed` | Execution | Execution(Dynamo) | artifact_ref, execution_fingerprint |
| `drift_signal` | Execution | Execution(Dynamo) | workflow_version_id, source_group, failure_rate |
| `admission_token_issued` | Admission | Egress(Dynamo) | token_id, attempt_id, pool_group |
| `egress_incident_detected` | Admission | Egress(Dynamo) | domain_group, incident_type |

## References

- RFC-004
- ADR-0002 (Outbox MUST)

