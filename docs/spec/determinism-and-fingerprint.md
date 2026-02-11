# Spec: Determinism, Canonicalization, and Execution Fingerprint

## Status

- Status: DRAFT → **NORMATIVE** (seeded from RFC-003/004, SOTA-refined)

## Purpose

- "같은 입력이면 같은 결과"의 정의를 **수학적 계약**으로 고정하고,
- 멱등 결과 재사용(idempotency)을 **사고 없이** 적용하기 위한 fingerprint/key 규칙을 단일화한다.
- Step Type 분류를 통해 Replay/Execution 경계를 **타입 시스템 수준**으로 강제한다.

## Core Contract (MUST)

Step Output는 아래 인자들의 **순수 함수**여야 한다.

```
step_output = f(step_version, canonical_input, task_context, capability_profile)
```

- `step_version`: 실행 가능한 스텝 스펙(코드/컨테이너/스키마/정책)의 버전화된 식별자
- `canonical_input`: 정규화된 입력 (canonical_spec_version 포함)
- `task_context`: 비결정 요소의 명시적 입력 (아래 상세)
- `capability_profile`: 외부 세계 접촉에 필요한 실행 프로필 (아래 상세)

**불변식**: 위 4개 인자가 동일하면 결과는 반드시 동일해야 한다. 이를 만족하지 못하는 step은 시스템에 등록 불가.

## Step Type System (MUST)

Workflow IR의 각 step은 **반드시** 아래 3개 타입 중 하나로 선언되어야 한다.

| Step Type | 외부 I/O | Replay 허용 | Admission 필요 | Fingerprint에 capability_profile 포함 |
|-----------|---------|------------|----------------|--------------------------------------|
| `Pure` | 금지 | 허용 | 불필요 | 불포함 (capability_profile_digest = NULL_HASH) |
| `Transform` | 금지 | 허용 | 불필요 | 불포함 (capability_profile_digest = NULL_HASH) |
| `Capability` | 필수 | **금지** | **필수** | **필수** |

- `Pure`: 입력만으로 출력이 결정되는 계산 (예: JSON 변환, 필터링, 집계)
- `Transform`: Raw 데이터를 입력받아 파생 데이터를 생성하는 변환 (예: HTML 파싱, 스키마 매핑)
- `Capability`: 외부 세계와 접촉하는 단계 (예: HTTP fetch, 브라우저 렌더링, 디바이스 제어)

**Replay 불변식**: Replay 모드에서는 `Capability` step의 Attempt 생성이 **구조적으로 차단**된다 (런타임 검사가 아니라 dispatch 레벨에서 거부).

## Canonicalization (MUST)

- `canonical_spec_version`은 WorkflowVersion(또는 Step 실행 컨텍스트)에 포함되어야 한다.
- `input_digest = hash(canonical_spec_version + canonical(input))`

### Canonicalization Rules (MUST)

1. JSON key 정렬: 재귀적 알파벳 순서
2. 공백/줄바꿈 정규화: compact JSON (no whitespace)
3. 숫자: IEEE 754 정규화 (trailing zero 제거)
4. null/undefined 구분: null은 명시, undefined/missing key는 제거
5. 배열 순서: 원본 순서 유지 (정렬하지 않음)

## Execution Fingerprint (MUST)

### Formal Definition

```
execution_fingerprint = hash(
    step_version
  + input_digest
  + canonical_spec_version
  + determinism_context_digest
  + capability_profile_digest
)
```

### Fingerprint 구성 요소 상세 (Closed List)

#### determinism_context_digest

`determinism_context_digest = hash(task_context.normalized)`

| 항목 | 수준 | 설명 |
|-----|------|------|
| `time_seed` | **MUST** | 시간 의존 로직의 결정론적 시드 |
| `random_seed` | **MUST** | 난수 의존 로직의 결정론적 시드 |
| `locale` | **MUST** | 문자열 정렬/포맷 규칙 |
| `timezone` | **SHOULD** | 시간대 의존 변환이 있는 경우 |
| `user_agent_class` | **SHOULD** | UA 의존 파싱이 있는 경우 |
| `accept_language` | **SHOULD** | 언어 의존 컨텐츠 선택이 있는 경우 |
| `rendering_profile` | **SHOULD** | 브라우저 렌더링 설정 의존이 있는 경우 |

**포함 금지 (MUST NOT)**: 실행 시각(wall clock), 워커 ID, 호스트명, 실행 순서 등 비결정적 런타임 값.

#### capability_profile_digest

`capability_profile_digest = hash(capability_profile.normalized)`

| 항목 | 수준 | 적용 조건 |
|-----|------|----------|
| `network_profile` | **MUST** | Capability step인 경우 |
| `egress_profile` | **MUST** | Capability step인 경우 |
| `proxy_pool_group` | **MUST** | 프록시 사용 시 |
| `device_profile` | **MUST** | 디바이스 워커 사용 시 |

**Pure/Transform step**: `capability_profile_digest = NULL_HASH` (상수). 외부 프로필이 결과에 영향을 줄 수 없으므로 fingerprint에서 사실상 제외.

### Fingerprint에 포함하지 않는 항목 (MUST NOT)

| 항목 | 이유 |
|------|------|
| `workflow_version` | 결과는 step 수준에서 결정됨. 워크플로우 수준 캐시 경계가 필요하면 `cache_scope` 옵션 사용 (ADR-0001) |
| `run_id` | 실행 인스턴스 식별자일 뿐 결과에 영향 없음 |
| `task_id` | 동상 |
| `attempt_id` | 동상 |
| `scheduled_at` / `started_at` | wall clock은 비결정적 |
| `worker_id` / `host` | 실행 위치는 결과에 영향 없어야 함 |

### IdempotencyRecord Key

- `idempotency_key = execution_fingerprint`

> **Cache Scope 확장 (ADR-0001)**: 기본값은 `workflow_version` **미포함**(강한 결과 재사용).
> `cache_scope=workflow_version` 옵션 시 `idempotency_key = hash(execution_fingerprint + workflow_version)`.

### Formal Properties (학계 SOTA 정합)

1. **결정론성(Determinism)**: 동일 fingerprint → 동일 결과 (by construction)
2. **충돌 저항(Collision Resistance)**: SHA-256 기반, 해시 충돌 확률 무시 가능
3. **분리성(Isolation)**: capability_profile 변경 → fingerprint 변경 → 캐시 miss (의도된 동작)
4. **단조성(Monotonicity)**: fingerprint는 한번 생성되면 변경 불가 (immutable)

## Replay Semantics (MUST)

### Replay 모드 규칙

1. `Capability` step의 Attempt 생성은 **구조적으로 금지** (dispatcher가 step_type 검사 후 거부)
2. `Pure`/`Transform` step만 실행 가능
3. Raw 데이터는 `raw-and-replay-index.md`의 Raw Selection Rule로 결정적 선택
4. Replay에서 생성된 결과도 idempotency store에 기록 (동일 fingerprint로)
5. Replay 결과와 원본 결과가 다르면 **drift_signal** 발행 (determinism 위반 탐지)

### Replay 불가 조건 (MUST FAIL)

- Raw Selection Rule로 raw_ref를 결정할 수 없는 경우
- 참조하는 upstream artifact가 존재하지 않는 경우
- step_type이 `Capability`인 step이 실행 경로에 포함된 경우

## References

- RFC-003
- RFC-004
- ADR-0001 (Idempotency Key Scope)

