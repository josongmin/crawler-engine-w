# Deployment Guide

## Purpose

- Control Plane / Execution Plane / Workers 배포 단위를 정의하고, 환경별(dev/stage/prod) 배포 원칙을 고정한다.

## Components

- Control Plane API (Definition Authority)
- Execution Plane API (Execution Authority)
- Orchestration layer (SFN/Temporal/Argo 등)
- Worker pools (ECS/Lambda/External/Device)
- Projectors (events → projections)

## Environments (TODO)

- dev: 로컬/샌드박스
- stage: 통합 테스트
- prod: 운영

## IaC (TODO)

- Terraform/CDK/CloudFormation 중 선택(ADR로)
- 최소 리소스 셋: RDS/Dynamo/S3/SQS/EventBridge + 관측 스택

