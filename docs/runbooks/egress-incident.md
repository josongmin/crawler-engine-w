# Runbook: Egress / Proxy Incident (Ban Storm, Capacity Drop)

## Purpose

- 밴 폭증/프록시 용량 저하/도메인 차단 등 egress 사고에서 확산을 멈추고 정상화한다.

## Preconditions

- 관련 스펙:
  - `../spec/admission-and-egress.md`
  - `../spec/policy-system.md`

## Safety Rules (MUST)

- Capability Step은 admission_token 없이는 실행 불가(스펙 준수 확인).
- 사고 시 신규 admission 발급을 우선 제한(throttle/deny)하고, 실행 중 작업은 정책에 따라 격리.

## Procedure

### 1) Identify

- 증상: 성공률 하락, 403/429 급증, timeout 급증, ban_rate 상승
- 범위: domain_group / proxy_pool_group / source_id

### 2) Contain

- 해당 pool 또는 domain_group admission 발급 제한
- schedule pause 또는 max_concurrency 하향
- 정책(user-agent/rendering/timeout/retry) 임시 완화/강화(ADR/승인 절차는 별도)

### 3) Remediate

- pool health 재평가
- cooldown window 적용
- 대체 pool/region 전환(가능한 경우)

### 4) Recover

- 점진적 트래픽 복구(토큰 발급 단계적 증가)
- ban_rate 정상화 확인 후 전체 재개

## Postmortem Notes

- 원인(도메인 정책/robots/트래픽 급증/프록시 품질/코드 변경)
- 재발 방지(정책 preset, admission 규칙, 관측 지표 추가)

