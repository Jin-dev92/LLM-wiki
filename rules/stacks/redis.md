---
type: rule
title: Redis 캐시 규칙 (Cluster)
summary: Redis Cluster 캐시 규칙 — 멀티키 연산은 해시태그로 슬롯 고정(CROSSSLOT 방지), SET/HSET 필수 TTL, mutation 시 관련 캐시 DEL, RedisModule DI로 연결 통합관리.
created: 2026-07-05
updated: 2026-07-05
visibility: team
scope: stack
applies_to: [redis, nestjs, prisma]
tags: [backend, redis, cache, rules]
source_file: 사용자 작성 Claude Code 시스템 프롬프트 cc-redis-rule.md (2026-07-05 반입, [[sources/backend/cc-redis-rule]])
---

## Redis 캐시 규칙 (Redis Cluster)

> 분산/Scale-out 환경에서 **Redis Cluster**를 전역 캐시로 쓰는 프로젝트 기준.
> 싱글 인스턴스 Redis를 가정한 코드(멀티키 연산 등)는 클러스터에서 그대로 깨진다.
> 관련: [[rules/stacks/nestjs]], [[rules/stacks/prisma]]

### CROSSSLOT 방지 (필수)

Redis Cluster는 키를 해시 슬롯 단위로 분산 저장한다. `MGET`/`MSET`/멀티키 `DEL`/파이프라인처럼
여러 키를 한 번에 다루는 연산은, 그 키들이 서로 다른 노드(슬롯)에 있으면 `CROSSSLOT` 에러로 실패한다.

연관된 키들은 **해시태그(`{}`)** 로 묶어 같은 슬롯에 강제 배치한다.

```ts
// BAD
redis.mget('user:100:profile', 'user:100:cart')

// GOOD
redis.mget('{user:100}:profile', '{user:100}:cart')
```

### TTL 필수 + 캐시 일관성

| 항목 | 규칙 |
|------|------|
| 캐시 쓰기 | `SET`/`HSET` 등에는 항상 명시적 TTL 포함(예: `'EX', 3600`). TTL 없는 캐시 쓰기 금지 |
| 캐시 무효화 | Prisma/TypeORM Mutation(Create/Update/Delete) 시 관련 캐시 키를 함께 `DEL` |

### 연결 관리

개별 비즈니스 서비스 안에서 Redis 클라이언트를 직접 `connect()`/`disconnect()`하지 않는다.
`PrismaService`와 동일한 패턴으로, NestJS DI로 주입되는 **단일 `RedisModule`**을 통해서만 접근한다.

### 코드 리뷰/생성 시 응답 규칙

요청받은 코드가 클러스터 비호환(예: 해시태그 없는 멀티키 연산)이면, 바로 고치지 말고 **먼저 문제를 짚어 경고한 뒤** 수정본을 제시한다.

---
