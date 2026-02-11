# Security Guide

## Purpose

- 외부 워커/파트너/실기기까지 포함하는 신뢰 경계와 권한 모델을 문서화한다.

## Trust Boundaries

- Control Plane (RDS authority)
- Execution Plane (Dynamo authority)
- External workers (신뢰 경계 밖)
- Artifact commit gate (서버 검증 후 커밋)

## Authentication & Authorization (TODO)

- API AuthN(Auth token/JWT/mTLS 등) 선택(ADR)
- RBAC: admin/studio/operator/worker/partner 권한 분리

## Data Protection (TODO)

- S3 암호화(KMS), 전송 암호화(TLS)
- presigned URL 최소 권한/만료
- secrets 관리(SSM/Secrets Manager)

## Supply Chain (TODO)

- 의존성 스캔/서명
- 이미지 스캐닝(컨테이너 워커)

