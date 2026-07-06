---
type: note
title: 벌크헤드 — 불안정 의존성의 동시성 격리
summary: 응답이 불안정한 외부 의존성이 이벤트 루프·커넥션 풀을 독점하지 못하게 동시 실행 수를 제한한다. Node(cockatiel)의 벌크헤드는 스레드 풀이 아니라 세마포어 방식. 수치는 실측 기반으로.
created: 2026-07-06
updated: 2026-07-06
visibility: private
provenance: extracted
tags: [resilience, bulkhead, backend, nodejs]
---

## 핵심
벌크헤드(격벽)는 배의 침수 구획처럼, 한 외부 의존성의 장애가 서비스 전체의 처리 용량을 잠식하지 못하도록 그 의존성에 쓸 수 있는 동시 실행 수를 상한으로 묶는 패턴이다.

## 상세
- **격리 대상**: 응답 시간이 불안정한 외부 API(결제·알림), 무거운 배치/리포트성 쿼리(일반 트래픽과 커넥션 풀 분리). 하나의 의존성이 스레드/이벤트 루프/커넥션 풀을 오래 점유할 수 있으면 격리한다.
- **수치는 실측으로**: 동시 실행 수는 대상 의존성의 실제 처리 능력(부하 테스트, 기존 트래픽 통계)을 근거로 정한다. 추측값 금지.
- **거절 처리**: 상한 초과로 거절되면(`BulkheadRejectedError`) 내부 에러를 그대로 노출하지 말고 사용자용 안내 메시지로 변환한다.
- **Node.js 주의**: 싱글 이벤트 루프라 cockatiel의 벌크헤드는 "동시 진행 개수 제한"(세마포어) 방식이다. Java(Resilience4j)의 스레드 풀 격리와 같은 것으로 오해하지 않는다.
- DB 커넥션 풀 분리가 필요하면 별도 `DataSource`/`extra.max` 설정으로 나눈다.

## 관련
- [[resilience-policy-composition-order]]
- [[redis-connection-management-via-di]] — 커넥션을 DI 싱글턴으로 통합 관리하는 것과 짝: 통합된 연결 위에 의존성별 사용량 상한을 거는 게 벌크헤드.
- [[rules/stacks/resilience]]
