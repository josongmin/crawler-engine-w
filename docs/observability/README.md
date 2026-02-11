# Observability Guide

## Purpose

- run_id / step / url_digest 기준으로 “단일 URL E2E” 추적이 가능해야 한다.

## Required Correlation Keys (MUST)

- `run_id`
- `step_id` and/or `step_version`
- `task_key`
- `attempt_id`
- `source_id`
- `url_digest`
- `execution_fingerprint`

## Signals (TODO)

- Metrics: 성공률, p95 latency, retry rate, idempotency hit ratio, ban_rate, admission token wait
- Logs: structured logs with correlation keys
- Traces: capability step 외부 호출은 span으로 분리

## Dashboards & Alerts (TODO)

- drift signal spike
- egress incident (403/429/timeout)
- cost anomaly

