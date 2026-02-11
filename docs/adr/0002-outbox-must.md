# ADR: Outbox Pattern MUST 여부 (Dispatch/Enqueue 경로)

## Status

- Date: 2026-02-09
- Status: PROPOSED
- Owners: TBD

## Context

외부 워커 경로에서 “큐 enqueue”와 “attempt 상태 갱신(Dynamo)”이 분리되면,
중복 enqueue / 유령 attempt / out-of-order 이벤트가 증가한다.

## Decision (proposal)

- Dispatch/Enqueue 같은 “부수효과 + 상태전이” 구간은 outbox를 **MUST**로 한다.
- Projector/dispatcher는 idempotent 해야 하며, 중복 이벤트는 무해화한다.

## Options Considered

1) SHOULD (권장)
2) MUST (proposal)
3) 엔진별 구현 선택(비권장)

## Consequences

- Positive: 상태/큐 정합성 논쟁 종결, 운영 코너케이스 감소
- Negative: 구현 복잡도 상승(아웃박스 테이블/프로세서 필요)
- Follow-ups:
  - outbox 스키마/재시도 규칙을 `../spec/authority-and-events.md`에 추가

## References

- `../spec/authority-and-events.md`
- `../spec/execution-semantics.md`
- RFC-004

