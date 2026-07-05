---
type: note
title: Redis 연결 관리는 DI 싱글턴으로 통합
summary: 개별 서비스에서 Redis connect/disconnect를 직접 하지 말고, PrismaService처럼 NestJS DI로 주입되는 단일 RedisModule을 통해서만 접근한다.
created: 2026-07-05
updated: 2026-07-05
visibility: private
provenance: extracted
tags: [redis, nestjs, di, backend]
---

## 핵심
비즈니스 서비스 코드 안에 Redis 클라이언트의 `connect()`/`disconnect()`를 직접 작성하지 않는다. 연결 수명주기는 프레임워크(NestJS) DI 컨테이너가 관리하는 단일 모듈에 맡긴다.

## 상세
이 패턴은 이미 `PrismaService`에서 쓰는 것과 동일하다 — DB 커넥션 풀을 서비스마다 새로 만들지 않고 싱글턴으로 주입받아 재사용하듯, Redis 클라이언트도 `RedisModule`(또는 동등한 provider)로 한 번만 생성해 필요한 곳에 주입한다. 이렇게 하면 커넥션 누수·중복 연결·종료 순서 문제를 개별 서비스가 아니라 프레임워크 라이프사이클에 위임할 수 있다.

## 관련
- [[rules/stacks/redis]]
- [[rules/stacks/nestjs]]
