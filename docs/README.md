# wine-crawler-engine Docs

이 폴더는 **실제 구현/운영을 위한 “규범(SSOT) 문서”** 모음이다.
RFC는 설계 논증/결정 기록이며, 본 `docs/`는 구현 시 참조하는 **스펙/런북/ADR**로 구조화한다.

## Navigation

- `../rfc/` : 설계 RFC (`005`가 Canon, `001~004`는 stub + archive)
- `INDEX.md` : 구현/운영 A to Z 전체 목차
- `COVERAGE.md` : 문서셋 빵꾸 점검 체크리스트
- `ROADMAP.md` : 문서/구현 로드맵(커버리지 기준)
- `spec/` : 안정화된 스펙(키/상태머신/이벤트/스토어/데이터레이아웃 등)
- `runbooks/` : 운영 런북(재처리, 장애, 백필, 비용/egress 등)
- `adr/` : Architecture Decision Records(작은 결정 단위)
- `templates/` : RFC/ADR/스펙 템플릿
  - `templates/RUNBOOK_TEMPLATE.md`

## Current Canon (v0)

초기 Canon은 `rfc/005`에 통합되어 있으며, 아래 스펙들이 1차 우선순위다.

- Execution Semantics (Run/Task/Attempt/Artifact, guards, invariants)
- Determinism & Idempotency (canonicalization, fingerprint, replay rules)
- Authority Split & Projection (event model 포함)
- RawIndex / Replay Lookup rules
- Admission Control / Egress SSOT

