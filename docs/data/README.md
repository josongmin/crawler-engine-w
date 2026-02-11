# Data Governance Guide

## Purpose

- Raw/Derived/Lineage/Indexes의 보관, 품질, 컴플라이언스(효과적 삭제)를 체계화한다.

## Data Zones

- Raw: immutable truth source
- Derived: transforms outputs, lineage required
- Lineage: computation graph SSOT
- Indexes: RawIndex, Idempotency records, projections

## Retention & Compliance (TODO)

- 데이터 등급 분류(PII/비PII)
- Effective delete 전략(ADR-0004 기반)
- 보관기간/아카이브/접근통제

## Quality (TODO)

- schema drift 신호
- 샘플링 검증

