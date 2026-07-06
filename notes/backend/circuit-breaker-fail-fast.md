---
type: note
title: 서킷 브레이커 — 계속 실패하는 의존성은 빠르게 차단
summary: 재시도로도 계속 실패하는 의존성엔 서킷 브레이커로 호출 자체를 차단(fail-fast)한다. 연속 실패 기반이 기본, halfOpenAfter는 실제 복구 시간 근거로. 상태 변화 로깅 필수 — 조용한 서킷은 금지.
created: 2026-07-06
updated: 2026-07-06
visibility: private
provenance: extracted
tags: [resilience, circuit-breaker, backend]
---

## 핵심
재시도가 "한 번 더 두드려 보는" 낙관이라면, 서킷 브레이커는 "지금은 두드려도 소용없다"는 판단이다. 실패가 누적된 의존성으로의 호출을 아예 차단(Open)해 즉시 실패시키고, 죽은 서버에 부하를 더하는 것과 우리 쪽 자원 낭비를 동시에 막는다.

## 상세
- **필수 적용 대상**: 결제/정산처럼 장애 시 사용자 영향이 큰 외부 연동, 응답 지연이 우리 전체 응답 시간에 직결되는 동기 호출.
- **Open 전환 기준**: 기본은 연속 실패 기반(`ConsecutiveBreaker`). 트래픽이 많은 의존성은 비율 기반 전환을 검토.
- **halfOpenAfter**(차단 후 시험 재개까지의 시간)는 대상 서비스의 평균 장애 복구 시간을 근거로 정한다. 근거 없는 임의값 금지.
- **에러 변환**: 차단 중 발생하는 `BrokenCircuitError`는 반드시 별도 캐치해 "일시적 장애 안내"로 변환한다.
- **관측 필수**: Open 진입/복구 등 상태 변화는 로그·모니터링(Sentry 등)에 남긴다. 조용히 실패하는 서킷 브레이커는 금지. cockatiel은 `onBreak`/`onReset`/`onHalfOpen` 이벤트 훅을 제공한다. *(inferred — 구체 API는 보강)*
- **fallback 설계**: 차단 시 안내 메시지가 전부가 아니다 — 캐시된 값 반환, 기본값, 나중 처리 큐잉 같은 graceful degradation을 의존성별로 검토한다. *(inferred — 원문에 없던 보강)*

## 관련
- [[retry-idempotency-and-backoff]] — 재시도의 각 시도가 브레이커의 실패 카운트에 집계된다(둘을 조합하면 Open 전환이 그만큼 빨라짐). *(inferred)*
- [[resilience-policy-composition-order]]
- [[rules/stacks/resilience]]
