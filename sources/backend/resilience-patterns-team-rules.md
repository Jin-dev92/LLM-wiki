---
type: source
title: 장애 대응 패턴 팀룰 (Retry·Bulkhead·Circuit Breaker)
summary: 사용자가 작성한 NestJS+cockatiel 기준 장애 대응(resilience) 팀룰 — 재시도 멱등성·지수백오프, 벌크헤드 격리, 서킷 브레이커, 정책 조합 순서, 코드 리뷰 체크리스트. Resilience4j 대응 규칙 부록 포함.
created: 2026-07-06
updated: 2026-07-06
visibility: private
url:
author: 김의진 (사용자 작성)
ingested_via: pdf
tags: [backend, resilience, nestjs, cockatiel, retry, circuit-breaker, bulkhead]
---

## 요약
외부 서비스 연동 코드(외부 API·내부 MSA 호출·실패 가능 I/O)에 적용할 장애 대응 패턴 3종의 팀룰. (1) 재시도는 멱등 요청 + 일시적 오류에만, 지수 백오프+Jitter 필수, (2) 벌크헤드로 불안정 의존성의 동시성 격리(Node에선 세마포어 방식), (3) 서킷 브레이커는 연속 실패 기반 기본 + 상태 변화 로깅 필수. 조합은 Timeout → Retry → CircuitBreaker → Bulkhead 순서로 `wrap()` 합성, 의존성별 정책 인스턴스 분리. PR 리뷰 체크리스트와 Spring(Resilience4j) 부록 포함.

## 반입 시 보강한 내용 (inferred)
원문 검토에서 발견한 공백을 위키화 과정에서 추가함 — 429의 `Retry-After` 존중, `wrap()` 인자 순서 의미(첫 인자 최외곽), Timeout 정책 코드(`timeout()`), 서킷 상태 이벤트 훅(`onBreak`/`onReset`/`onHalfOpen`), `BrokenCircuitError` fallback 전략(캐시값·기본값·큐잉), Retry 각 시도가 브레이커 실패 카운트에 집계된다는 상호작용. 오타 1건(임의值→임의값) 정정.

## 추출한 영구노트
- [[retry-idempotency-and-backoff]]
- [[bulkhead-semaphore-isolation]]
- [[circuit-breaker-fail-fast]]
- [[resilience-policy-composition-order]]

## 출처 원문 메모
로컬 파일 `~/Downloads/resilience-patterns-team-rules.md`(사용자 작성, 2026-07-06 위키 반입). 원문은 [[rules/stacks/resilience]]로 정제되어 팀 공유용 규칙(scope: stack)으로 편입됨.
