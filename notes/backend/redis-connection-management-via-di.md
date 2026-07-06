---
type: note
title: Redis 연결 관리는 DI 싱글턴으로 통합
summary: 개별 서비스에서 Redis connect/disconnect를 직접 하지 말고, PrismaService처럼 NestJS DI로 주입되는 단일 RedisModule을 통해서만 접근한다.
created: 2026-07-05
updated: 2026-07-06
visibility: private
provenance: extracted
tags: [redis, nestjs, di, backend, pubsub]
---

## 핵심
비즈니스 서비스 코드 안에 Redis 클라이언트의 `connect()`/`disconnect()`를 직접 작성하지 않는다. 연결 수명주기는 프레임워크(NestJS) DI 컨테이너가 관리하는 단일 모듈에 맡긴다.

## 상세
이 패턴은 이미 `PrismaService`에서 쓰는 것과 동일하다 — DB 커넥션 풀을 서비스마다 새로 만들지 않고 싱글턴으로 주입받아 재사용하듯, Redis 클라이언트도 `RedisModule`(또는 동등한 provider)로 한 번만 생성해 필요한 곳에 주입한다. 이렇게 하면 커넥션 누수·중복 연결·종료 순서 문제를 개별 서비스가 아니라 프레임워크 라이프사이클에 위임할 수 있다.

## 예외: pub/sub 전용 커넥션은 직접 정리한다
Redis 구독(subscribe) 모드 연결은 일반 명령을 못 쓰는 제약 때문에, 단일 클라이언트를 쓰지 못하고 `redis.duplicate()`로 **전용 커넥션**을 별도로 만드는 것이 정당하다. 문제는 이 커넥션이 DI 싱글턴 밖에 있어 프레임워크가 자동으로 정리해 주지 않는다는 점이다. 따라서 이 커넥션을 만든 서비스가 `OnModuleDestroy`(또는 동등한 종료 훅)에서 직접 `quit()`을 호출해 연결 누수를 막아야 한다. "직접 만든 연결은 직접 닫는다"가 위 원칙의 유일한 예외다.

## 관련
- [[rules/stacks/redis]]
- [[rules/stacks/nestjs]]
- [[bulkhead-semaphore-isolation]] — 통합된 연결 위에 의존성별 동시 사용량 상한을 거는 게 벌크헤드.
