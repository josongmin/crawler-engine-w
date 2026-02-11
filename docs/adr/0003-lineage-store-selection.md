# ADR: Lineage Store 선택 (GraphDB vs RDB vs Dynamo)

## Status

- Date: 2026-02-09
- Status: PROPOSED
- Owners: TBD

## Context

Lineage는 영향 분석/부분 재계산/감사에 핵심이며, 저장소 선택이 쿼리 성능과 운영 복잡도를 좌우한다.

## Decision (proposal)

- 1차 목표는 “필수 쿼리 패턴”을 만족하는 최소 구현을 빠르게 확보한다.
- 후보:
  - RDB adjacency + index (운영 단순, 쿼리 예측 가능)
  - GraphDB (탐색 풍부, 운영/비용 부담)
  - Dynamo single-table (비용/성능 좋을 수 있으나 쿼리 제한)

## Options Considered

1) RDB adjacency + index
2) GraphDB
3) Dynamo single-table

## Consequences

- Positive: (선택 이후 작성)
- Negative: (선택 이후 작성)
- Follow-ups:
  - `../spec/lineage-graph.md`의 Storage 섹션을 확정

## References

- `../spec/lineage-graph.md`
- RFC-003

