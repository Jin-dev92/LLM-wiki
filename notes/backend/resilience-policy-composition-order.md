---
type: note
title: 장애 대응 정책 조합 순서 (Timeout→Retry→CB→Bulkhead)
summary: 외부 의존성 호출은 Timeout → Retry → CircuitBreaker → Bulkhead 순으로 wrap 합성한 단일 정책으로 감싼다. 정책 인스턴스는 의존성별로 분리 — 서킷 상태가 섞이면 안 된다.
created: 2026-07-06
updated: 2026-07-06
visibility: private
provenance: extracted
tags: [resilience, backend, cockatiel]
---

## 핵심
Retry·Bulkhead·CircuitBreaker는 낱개로 흩어 쓰는 게 아니라, 정해진 순서로 합성한 단일 정책을 의존성마다 하나씩 만들어 재사용한다.

## 상세
- **순서**: `Timeout → Retry → CircuitBreaker → (전체를) Bulkhead로 격리`. cockatiel에선 `wrap(retryPolicy, circuitBreakerPolicy, bulkheadPolicy)`처럼 합성한다.
- `wrap()`은 **첫 인자가 최외곽**이다 — `wrap(retry, cb, bulkhead)`는 재시도가 서킷 브레이커를 감싸는 구조라서, 각 재시도 시도가 브레이커에 개별 실패로 집계되고 도중에 Open되면 남은 재시도가 즉시 차단된다. 순서를 바꾸면 이 동작이 달라지므로 임의 변경 금지. *(inferred — 원문에 없던 보강)*
- **의존성별 인스턴스 분리**: 정책 조합은 연동 대상별로 하나씩 정의한다. 여러 의존성이 정책 인스턴스를 공유하면 A 서비스 장애로 열린 서킷이 B 서비스 호출까지 차단하는 오작동이 된다.
- **상수화**: 합성 정책은 서비스 클래스에 상수로 선언해 재사용한다. 매 메서드/호출마다 새로 생성하면 서킷 브레이커가 상태(실패 카운트)를 누적하지 못해 무력화된다.

## 관련
- [[retry-idempotency-and-backoff]]
- [[bulkhead-semaphore-isolation]]
- [[circuit-breaker-fail-fast]]
- [[rules/stacks/resilience]]
