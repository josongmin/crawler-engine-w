# Spec: Control Plane API

## Status

- Status: DRAFT
- Authority: RDS (Definition Authority)

## Purpose

- WorkflowDefinition/Version/Policy/Schedule을 관리하는 API 계약을 고정한다.

## Resources (minimum)

- Workflow
- WorkflowVersion (immutable after publish)
- Schedule
- Source
- StepCatalog
- PolicyPreset / PolicyOverride

## Normative Rules (MUST)

- Publish는 RDS 트랜잭션 내부에서만 완료
- Publish 이후에만 `workflow_version_published` 이벤트 발행
- Publish된 workflow_version은 수정 불가(새 버전만)

## Endpoints (skeleton)

### Workflows

- `POST /workflows`
- `GET /workflows/{workflow_id}`
- `LIST /workflows`

### Workflow Versions

- `POST /workflows/{workflow_id}/versions` (create draft)
- `GET /workflow_versions/{workflow_version}`
- `POST /workflow_versions/{workflow_version}/publish`
- `POST /workflow_versions/{workflow_version}/rollback` (reactivate old)

### Schedules

- `POST /schedules`
- `GET /schedules/{schedule_id}`
- `POST /schedules/{schedule_id}/pause`
- `POST /schedules/{schedule_id}/resume`

### Policy

- `GET /policy_presets`
- `POST /policy_presets`
- `POST /sources/{source_id}/policy_override`

## Validation (MUST)

- Workflow DSL/IR 정적 검증은 publish 이전에 통과해야 함
- fan-out 상한, sink 없는 흐름, 과도 retry 등 lint 규칙 적용

## References

- RFC-002
- RFC-003
- RFC-004

