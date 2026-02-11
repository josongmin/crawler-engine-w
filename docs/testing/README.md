# Testing Guide

## Purpose

- 상태머신/결정론/멱등/리플레이/외부 워커 커밋 게이트를 “테스트로 증명”한다.

## Test Layers (TODO)

- Unit: canonicalization, fingerprint, raw selection rules
- State-machine property tests: 단조성/단일 active attempt/invariants
- Integration: Dynamo conditional writes, outbox projector, presigned upload commit
- E2E(staging): SQS fan-out + worker pool + replay

## Required Test Matrix (MUST)

- duplicate dispatch
- out-of-order events
- lease timeout / lost worker
- commit failure after “success report”
- replay with missing raw (expected fail)

