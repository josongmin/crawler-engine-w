# Spec: Admission Control and Egress SSOT

## Status

- Status: DRAFT → **NORMATIVE** (seeded from RFC-004, SOTA-refined)

## Purpose

- proxy/IP/ban/capacity 같은 "물리 제약"을 시스템 의미론에 포함한다.
- 외부 접속(Capability Step)이 egress 토큰 없이 실행되는 것을 금지한다.
- 토큰 발급/반납/만료 모델을 형식화하여 리소스 누수를 구조적으로 방지한다.

## Normative Rules (MUST)

1. Capability Step은 `admission_token` 없이 **실행 불가**
2. Admission Control은 Capability Step 앞에서 수행되어야 함
3. Token 없이 dispatch된 Capability Attempt는 즉시 **ATTEMPT_FAILED** (admission_violation)

## Admission Token Model (MUST)

### 발급 단위

- **Attempt 단위**: 하나의 Attempt에 하나의 `admission_token`
- token과 attempt는 1:1 바인딩 (token 재사용 금지)

### Token Lifecycle

```
[요청] → [발급] → [활성] → [반납 또는 만료]
```

| State | 설명 |
|-------|------|
| `TOKEN_REQUESTED` | Attempt가 admission 요청 |
| `TOKEN_ISSUED` | 토큰 발급 완료, attempt에 바인딩 |
| `TOKEN_ACTIVE` | 워커가 실행 중 (heartbeat와 연동) |
| `TOKEN_RETURNED` | Attempt 종료 시 정상 반납 |
| `TOKEN_EXPIRED` | TTL 초과 시 자동 회수 |

### 반납/만료 규칙 (MUST)

| 트리거 | 동작 |
|--------|------|
| `ATTEMPT_SUCCEEDED` | token 즉시 반납 → capacity 복원 |
| `ATTEMPT_FAILED` | token 즉시 반납 → capacity 복원 |
| `ATTEMPT_LEASE_EXPIRED` | token TTL 만료 → 자동 회수 |
| Token TTL 초과 | `lease_duration` 연동. TTL = `lease_duration + grace_period(30초)` |
| 미반납 감지 | Reaper가 만료 token 회수, capacity 복원, alert 발행 |

### 발급 제어 차원 (MUST)

Token 발급은 아래 3개 차원의 **AND 조건**으로 제어된다:

| 차원 | Key | 설명 |
|------|-----|------|
| Domain Group | `domain_group_id` | 대상 도메인에 대한 동시 접속 제한 (politeness) |
| Proxy Pool Group | `proxy_pool_group_id` | 프록시 풀의 가용 capacity |
| Global Budget | `global_budget` | 시스템 전체 동시 egress 상한 |

### Domain Politeness Integration (MUST)

- `robots.txt` crawl-delay는 `domain_group` 수준의 **rate limit**으로 반영
- admission은 rate limit과 capacity limit 중 **더 엄격한 쪽**을 적용

## Egress SSOT (Authority)

- Authority: **DynamoDB**

### Entities (MUST)

| Entity | 주요 필드 | 역할 |
|--------|----------|------|
| `ProxyPoolGroup` | pool_id, capacity, active_tokens, health_score, ban_rate, region, ASN | 프록시 풀 상태 |
| `EgressProfile` | profile_id, network_type, proxy_preference, geo_constraint | 실행 프로필 |
| `IPState` | ip, pool_id, status(healthy/banned/cooldown), last_used, ban_count | IP 단위 상태 |
| `BanHistory` | ip, domain_group, banned_at, ban_reason, detected_by | ban 이력 |
| `CooldownWindow` | domain_group, pool_id, cooldown_until, reason | 쿨다운 상태 |
| `AdmissionToken` | token_id, attempt_id, domain_group_id, pool_group_id, issued_at, ttl, state | 토큰 상태 |

### Capacity Accounting (MUST)

- `ProxyPoolGroup.active_tokens`는 발급/반납/만료 시 **conditional update**로 원자적 증감
- capacity = `ProxyPoolGroup.capacity - ProxyPoolGroup.active_tokens`
- capacity <= 0 이면 발급 거부 (backpressure)

## Egress Incident Response (MUST)

| 단계 | 동작 |
|------|------|
| 감지 | ban_rate 임계치 초과 또는 health_score 하락 → `egress_incident_detected` 이벤트 |
| 격리 | 해당 domain_group/pool_group의 admission 발급 **즉시 중단** |
| 완화 | cooldown 적용 + 대체 pool 탐색 |
| 복구 | 점진적 admission 재개 (canary → ramp-up) |

## References

- RFC-004
- `spec/execution-semantics.md` (lease/attempt 연동)
- `runbooks/egress-incident.md` (운영 절차)

