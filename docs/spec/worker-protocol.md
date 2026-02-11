# Spec: Worker Protocol (Internal/External/Device)

## Status

- Status: DRAFT → **NORMATIVE** (SOTA-refined)
- Depends on: `execution-semantics.md`, `determinism-and-fingerprint.md`, `admission-and-egress.md`

## Purpose

- 워커(내부 ECS/Lambda, 외부 파트너, 실기기)의 실행/업로드/커밋 프로토콜을 단일 계약으로 고정한다.

## Worker Types (MUST)

| Type | 실행 환경 | Lease 방식 | 신뢰 경계 |
|------|----------|-----------|----------|
| **Internal** | ECS Fargate / Lambda | 서버가 lease 발급 → push dispatch | 신뢰 경계 내 |
| **External** | 파트너 인프라 | Pull 기반 (SQS poll) + commit protocol | **신뢰 경계 밖** |
| **Device** | 실제 디바이스 (모바일/IoT) | Pull 기반 + device_profile 필수 | **신뢰 경계 밖** |

> **신뢰 경계 원칙**: External/Device 워커의 산출물은 무조건 서버 측 검증(Commit Gate)을 통과해야 한다. Internal 워커도 동일 프로토콜을 따르되, 검증 수준을 정책으로 조정 가능.

## Capability Profile (MUST)

워커는 자신의 capability를 선언하고, 실행은 `capability_profile_digest`로 식별된다.

### Capability Profile Schema (Normative)

```json
{
  "network_profile": {
    "type": "direct | proxy | vpn",
    "geo": "country_code",
    "asn": "optional"
  },
  "egress_profile": {
    "proxy_pool_group": "pool_group_id",
    "ip_rotation": "per_request | per_session | static",
    "protocol": "http | https | socks5"
  },
  "device_profile": {
    "device_type": "desktop | mobile | tablet | iot",
    "os": "string",
    "browser": "string | null",
    "rendering_engine": "chromium | webkit | none",
    "viewport": "WxH | null"
  },
  "resource_profile": {
    "cpu_class": "small | medium | large",
    "memory_mb": "integer",
    "timeout_sec": "integer",
    "gpu": "boolean"
  }
}
```

### Profile Normalization Rule (MUST)

- `capability_profile_digest = hash(canonical(capability_profile))`
- 정규화 규칙은 `determinism-and-fingerprint.md`의 Canonicalization Rules를 따름
- Pure/Transform step에서는 `capability_profile_digest = NULL_HASH`

## Pull / Lease Model (MUST)

### Leasing (MUST)

- Lease 권위자: **Execution Authority(DynamoDB)** 단일
- Lease 생성: `ATTEMPT_LEASED` 전이 시 conditional write로 `lease_expiry` 기록
- 단일 소유권: conditional write 조건 = "해당 Task에 active attempt 없음"
- Heartbeat: `lease_duration / 3` 간격, 서버 시간 기준
- 만료: `lease_expiry < now() - 5초(clock skew)` → `ATTEMPT_LEASE_EXPIRED`
- 상세 규칙: `execution-semantics.md` Lease Model 섹션 참조

### External Dispatch (SQS) (MUST)

- enqueue + 상태전이는 **Dynamo outbox로 원자성 확보** (ADR-0002 MUST)
- SQS 메시지에 `attempt_id` + `lease_expiry` 포함
- 워커는 SQS 수신 후 heartbeat 시작 전에 lease 유효성 확인

## Artifact Commit Protocol (MUST)

### 전체 흐름

```
Worker                          Server
  |  1. presigned URL 요청         |
  |  <---- presigned URL ------   |
  |  2. S3 직접 업로드              |
  |  3. commit_artifact 호출  ---> |
  |     (artifact_digest, meta)    |
  |                    4. S3 head 확인 → STAGED
  |                    5. 검증 실행
  |                    6. 통과 → COMMITTED / 실패 → REJECTED
  |  <---- commit 결과 반환 -----  |
```

### Commit Verification Rules (MUST)

서버는 아래 **모든** 검증을 통과한 경우에만 `ARTIFACT_COMMITTED`로 확정한다.

| # | 검증 항목 | 실패 시 |
|---|----------|--------|
| V1 | **Hash 무결성**: 워커가 제출한 `artifact_digest` == 서버가 계산한 S3 object digest | `ARTIFACT_REJECTED` (integrity_mismatch) |
| V2 | **Size 일치**: 제출 size == S3 object size | `ARTIFACT_REJECTED` (size_mismatch) |
| V3 | **Manifest 존재**: raw artifact인 경우 manifest 파일이 동반 업로드됨 | `ARTIFACT_REJECTED` (manifest_missing) |
| V4 | **Schema 적합성**: output_schema가 정의된 step인 경우, 산출물이 schema에 적합 | `ARTIFACT_REJECTED` (schema_violation) |
| V5 | **Policy 적합성**: 금지된 컨텐츠/포맷/크기 정책 위반 없음 | `ARTIFACT_REJECTED` (policy_violation) |

### Commit Timing Constraint (MUST)

- `commit_artifact` 호출은 `lease_expiry` 이전에 도달해야 함
- lease 만료 후 도착한 commit 요청 → conditional write 실패 → **무시** (attempt는 이미 EXPIRED)

### Attempt → Artifact 연동 (MUST)

- `ARTIFACT_COMMITTED` → Attempt를 `ATTEMPT_SUCCEEDED`로 전이
- `ARTIFACT_REJECTED` → Attempt를 `ATTEMPT_FAILED`로 전이 (verify_failure reason)
- 하나의 Attempt는 하나의 primary artifact만 commit 가능 (1:1)

## Error Reporting (MUST)

### Error Payload Schema

```json
{
  "error_class": "TRANSIENT | PERMANENT | UNKNOWN",
  "retryable": true | false,
  "category": "network | parse | validation | resource | internal",
  "message": "human-readable description",
  "stack": "optional stack trace",
  "metadata": {}
}
```

### Error Handling Rules (MUST)

- out-of-order/duplicate 보고는 conditional write로 **무해화**
- lease 만료 후 도착한 error 보고 → **무시** (attempt 이미 EXPIRED)
- `error_class=PERMANENT` → retry 없이 즉시 `ATTEMPT_FAILED`
- `error_class=TRANSIENT` + `retryable=true` → retry 정책에 따라 새 Attempt 가능

## References

- RFC-001
- RFC-002
- RFC-004
- ADR-0002 (Outbox MUST)
- `spec/execution-semantics.md` (Lease Model, State Transitions)
- `spec/determinism-and-fingerprint.md` (Capability Profile Digest)
- `spec/admission-and-egress.md` (Admission Token)

