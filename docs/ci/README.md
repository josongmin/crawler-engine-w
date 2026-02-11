# CI Guide

## Purpose

- 스펙/코드/인프라 변경이 “재현성/정합성/보안”을 깨지 않도록 자동 검증한다.

## Pipelines (TODO)

- Lint/Format
- Unit tests
- Integration tests (Dynamo/S3/SQS/StepFunctions 모킹 또는 staging)
- Spec consistency checks (키/상태머신 closed set 변경 감지)

## Release Gates (TODO)

- publish validation/lint 규칙과 CI 연동
- drift signal/egress incident에 대한 smoke test

