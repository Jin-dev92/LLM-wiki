---
type: rule
title: 장애 대응 패턴 규칙 (Retry·Bulkhead·Circuit Breaker)
summary: 외부 의존성 호출의 장애 대응 규칙 — 멱등+일시적 오류에만 재시도(지수백오프+Jitter), 벌크헤드 동시성 격리, 서킷 브레이커(상태 로깅 필수), Timeout→Retry→CB→Bulkhead 조합. NestJS+cockatiel 기준, Resilience4j 부록.
created: 2026-07-06
updated: 2026-07-06
visibility: team
scope: stack
applies_to: [nestjs, cockatiel, resilience4j, spring]
tags: [backend, resilience, retry, circuit-breaker, bulkhead, rules]
source_file: 사용자 작성 팀룰 resilience-patterns-team-rules.md (2026-07-06 반입, [[sources/backend/resilience-patterns-team-rules]]) + 반입 시 보강(429 Retry-After·wrap 순서·timeout 코드·이벤트 훅·fallback)
---

## 장애 대응 패턴 규칙 (Retry / Bulkhead / Circuit Breaker)

> NestJS + **cockatiel** 기준. Spring(Resilience4j) 대응은 부록 참조.
> 관련: [[rules/stacks/nestjs]], [[rules/stacks/redis]]

### 적용 범위

다음 코드에는 이 규칙을 **필수** 적용한다. 내부 함수 호출·순수 계산 로직은 제외.
- 외부 API 호출(결제사·알림·서드파티 연동)
- 내부 마이크로서비스 간 호출
- 실패 가능성이 있는 I/O(DB·캐시·메시지 큐)

### Retry (재시도)

**걸기 전 확인(둘 다 통과해야 함):**

| 확인 | 기준 |
|------|------|
| 멱등성 | `GET`/`PUT`/`DELETE` O, `POST` 기본 X. POST 재시도가 필요하면 `Idempotency-Key` 헤더 발급 + 서버의 동일 키 1회 처리 보장 확인 |
| 일시성 | 재시도 O: 타임아웃·연결 끊김·`429`·`500/502/503/504` / 재시도 X: `400/401/403/404` — 4xx 재시도 PR은 반려 |

**구현 규칙:**
- 최대 시도 횟수 명시(기본 3회) + 지수 백오프 + Jitter. 고정 간격 금지.
- 전체 재시도 과정에 타임아웃 예산을 둔다.
- `429`에 `Retry-After` 헤더가 있으면 백오프 계산보다 우선 존중한다.
- 재시도 정책은 서비스별 상수화해 재사용(매 메서드 생성 금지). `handleAll` 대신 가능하면 `handleType()`으로 대상 예외를 좁힌다.

```typescript
const retryPolicy = retry(handleAll, {
  maxAttempts: 3,
  backoff: new ExponentialBackoff(),
});
```

### Bulkhead (동시성 격리)

- 격리 대상: 응답 시간이 불안정한 외부 API(결제·알림), 무거운 배치/리포트성 쿼리(일반 트래픽과 커넥션 풀 분리). 이벤트 루프·커넥션 풀을 오래 점유할 수 있는 의존성.
- 동시 실행 수는 **실측 근거**(부하 테스트·트래픽 통계)로 정한다. 추측값 금지.
- `BulkheadRejectedError`는 캐치해 사용자용 안내 메시지로 변환한다(내부 에러 노출 금지).
- DB 커넥션 풀 분리는 별도 `DataSource`/`extra.max` 설정으로.
- Node.js는 싱글 이벤트 루프 — cockatiel 벌크헤드는 **세마포어 방식**(동시 진행 개수 제한)이다. 스레드 풀 격리로 오해하지 않는다.

```typescript
// bulkhead(동시 최대 실행 수, 대기열 크기)
const paymentBulkhead = bulkhead(5, 10);
```

### Circuit Breaker (차단)

- 필수 대상: 결제/정산 등 장애 시 사용자 영향이 큰 외부 연동, 응답 지연이 전체 응답 시간에 직결되는 동기 호출.
- Open 전환 기준은 기본 **연속 실패 기반**(`ConsecutiveBreaker`), 트래픽 많은 의존성은 비율 기반 검토.
- `halfOpenAfter`는 대상 서비스의 평균 장애 복구 시간을 근거로 정한다(근거 없는 임의값 금지).
- `BrokenCircuitError`는 별도 캐치해 "일시적 장애 안내"로 변환하고, 가능하면 의존성별 **fallback**(캐시된 값·기본값·나중 처리 큐잉)을 함께 설계한다.
- 상태 변화(Open 진입/복구)는 **반드시 로깅/모니터링**(Sentry 등)한다. cockatiel의 `onBreak`/`onReset`/`onHalfOpen` 훅 사용. 조용히 실패하는 서킷 금지.

```typescript
const circuitBreakerPolicy = circuitBreaker(handleAll, {
  halfOpenAfter: 10 * 1000,
  breaker: new ConsecutiveBreaker(5),
});
circuitBreakerPolicy.onBreak(() => logger.warn('circuit OPEN: payment'));
circuitBreakerPolicy.onReset(() => logger.log('circuit CLOSED: payment'));
```

### 조합 (필수 순서)

```
Timeout → Retry → CircuitBreaker → (전체를) Bulkhead로 격리
```

```typescript
const timeoutPolicy = timeout(2000, TimeoutStrategy.Aggressive); // 시도당 타임아웃
const resilientPolicy = wrap(retryPolicy, circuitBreakerPolicy, bulkheadPolicy);
```

- `wrap()`은 **첫 인자가 최외곽**: `wrap(retry, cb, bulkhead)` = 재시도가 서킷을 감싼다 → 각 재시도 시도가 브레이커 실패 카운트에 개별 집계되고, 도중 Open되면 남은 재시도가 즉시 차단된다. 순서 임의 변경 금지.
- 합성 정책은 서비스 클래스에 상수로 선언해 재사용한다.
- 정책 조합은 **의존성별로 하나씩**. 인스턴스를 공유하면 서킷 상태가 섞여 무관한 의존성까지 차단된다.

### 코드 리뷰 체크리스트

PR에 외부 의존성 호출이 포함되면:
- [ ] 비멱등 요청(`POST` 등)에 재시도가 없거나, 있다면 `Idempotency-Key`가 있는가
- [ ] 4xx에 재시도가 걸려 있지 않은가
- [ ] 재시도에 최대 횟수 + 지수 백오프(+Jitter)가 있는가
- [ ] 응답 지연 가능 의존성에 벌크헤드가 있는가
- [ ] 사용자 영향 큰 의존성에 서킷 브레이커가 있는가
- [ ] `BulkheadRejectedError`/`BrokenCircuitError`가 각각 별도 처리되어 사용자 메시지로 변환되는가
- [ ] 서킷 상태 변화가 로깅/모니터링되는가

### 부록: Spring Boot (Resilience4j)

```java
@CircuitBreaker(name = "paymentService", fallbackMethod = "fallbackCharge")
@Retry(name = "paymentService")
@Bulkhead(name = "paymentService")
public PaymentResult chargePayment(String orderId, BigDecimal amount) { ... }
```

- 파라미터는 코드가 아닌 `application.yml`에서 관리.
- `name`은 의존성 단위로 고유 부여, 서비스 전체 공유 금지.

---
