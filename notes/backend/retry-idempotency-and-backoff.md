---
type: note
title: 재시도는 멱등성과 일시성이 전제 (지수 백오프+Jitter)
summary: 재시도는 멱등 요청 + 일시적 오류(타임아웃·429·5xx)에만 건다. POST는 Idempotency-Key 없이 재시도 금지, 4xx 재시도는 반려. 고정 간격 대신 지수 백오프+Jitter, 전체 타임아웃 예산 필수.
created: 2026-07-06
updated: 2026-07-06
visibility: private
provenance: extracted
tags: [resilience, retry, backend, http]
---

## 핵심
재시도를 걸기 전에 두 가지를 먼저 물어야 한다 — "같은 요청을 두 번 보내도 안전한가(멱등성)"와 "다시 시도하면 성공할 수 있는 실패인가(일시성)". 둘 중 하나라도 아니면 재시도는 장애를 키우는 코드다.

## 상세
- **멱등성**: `GET`/`PUT`/`DELETE`는 기본 멱등, `POST`는 아님. POST를 재시도해야 하면 `Idempotency-Key` 헤더를 발급하고 서버가 동일 키를 1회만 처리하는지 확인한다. 이 확인 없는 POST 재시도는 결제 중복 같은 사고로 직결된다.
- **일시성**: 타임아웃·연결 끊김·`429`·`500/502/503/504`만 재시도 대상. `400/401/403/404`는 다시 보내도 같은 결과이므로 재시도를 걸면 안 되며, 걸려 있는 PR은 반려한다.
- **간격**: 고정 간격 재시도는 금지 — 장애 난 서버에 동시 재시도가 몰리는 thundering herd를 만든다. 지수 백오프(Exponential Backoff)에 Jitter(무작위 편차)를 더해 분산시킨다.
- **예산**: 최대 시도 횟수(기본 3회)와 전체 재시도 과정의 타임아웃 예산을 명시한다. 사용자를 무한정 기다리게 하지 않는다.
- `429` 응답에 `Retry-After` 헤더가 있으면 백오프 계산보다 그 값을 우선 존중한다. *(inferred — 원문에 없던 보강)*

## 관련
- [[resilience-policy-composition-order]]
- [[circuit-breaker-fail-fast]]
- [[rules/stacks/resilience]]
