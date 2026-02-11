# Runbook: <TITLE>

## Purpose

- 어떤 상황에서 이 런북을 실행하는가?
- 성공/종료 조건은 무엇인가?

## Preconditions

- 필요한 권한/도구/대상 환경
- 관련 스펙 링크(필수)

## Safety Rules (MUST)

- 되돌릴 수 없는 동작 금지/주의
- 데이터 손상 방지 체크

## Procedure

### 1) Identify

- 증상/알람/대시보드
- 범위 파악(run_id/source_id/domain_group 등)

### 2) Contain

- 확산 방지(egress 차단, admission throttle, schedule pause 등)

### 3) Remediate

- 조치 단계
- 단계별 검증

### 4) Recover

- 재실행/리플레이/백필
- 정상화 확인

### 5) Postmortem Notes

- 원인 가설
- 재발 방지 액션(스펙/코드/운영)

## References

- Spec:
- RFC:
- ADR:

