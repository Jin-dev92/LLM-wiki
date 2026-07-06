---
type: rule
title: Redis 캐시 규칙 (Cluster)
summary: Redis Cluster 캐시 규칙 — 멀티키 연산은 해시태그로 슬롯 고정(CROSSSLOT 방지), SET/HSET·capped 리스트 필수 TTL, mutation 시 관련 캐시 DEL, 파생 카운터(DB COUNT 폴백) 패턴, RedisModule DI 및 pub/sub duplicate 커넥션 정리.
created: 2026-07-05
updated: 2026-07-06
visibility: team
scope: stack
applies_to: [redis, nestjs, prisma]
tags: [backend, redis, cache, rules]
source_file: 사용자 작성 Claude Code 시스템 프롬프트 cc-redis-rule.md (2026-07-05 반입, [[sources/backend/cc-redis-rule]]); estate-server CLAUDE.md Redis 캐시 규칙 (2026-07-06 파생 카운터·capped TTL·pub/sub 정리 수확)
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
| capped 리스트 | `LPUSH`+`LTRIM`로 최근 N개만 유지하는 리스트도 캐시 쓰기다. `LTRIM`은 길이만 자를 뿐 키를 만료시키지 않으므로 `EXPIRE`를 함께 걸어 무활동 키 잔존을 막는다 |
| 캐시 무효화 | Prisma/TypeORM Mutation(Create/Update/Delete) 시 관련 캐시 키를 함께 `DEL` |

#### 파생 카운터(derived counter) 패턴

미읽음 수·좋아요 수처럼 **DB 집계(`COUNT(*)`)의 파생 캐시**인 카운터는 TTL 없는 "영속 카운터"로 두지 않는다.
진실 원천(source of truth)은 DB이고 Redis는 그 캐시라는 원칙 아래 다음 패턴을 따르면, TTL 규칙을 지키면서도 캐시 유실이 영구화되지 않는다.

- **증감은 키가 존재할 때만**(Lua `EXISTS` 가드). 미스 키에 `INCR`하면 `0→1`로 실제 카운트를 잃으므로 건드리지 않고, 재구축은 읽기 경로에 맡긴다.
- **읽기는 "카운터 우선 → 미스면 DB `COUNT` → `SET NX EX` 백필"**. `get`은 미스를 `null`로 신호해 값 `0`과 구분한다.
- **TTL은 백필만 소유**한다(증감 시 갱신 안 함). 그래야 TTL이 drift(캐시-DB 불일치)의 상한(hard ceiling)이 된다.
- 리셋(전건 읽음 등)은 값을 `0`으로 덮기보다 **키 삭제(`DEL`)**가 안전하다 — 다음 읽기가 DB로 재집계하므로 리셋과 동시 증감이 겹치는 경합을 자가 교정한다.

> 한계: 미스 재구축 중 새 증감이 끼면 최대 TTL만큼 ±1 drift가 남는 best-effort다. TTL이 그 상한 역할을 한다.

### 연결 관리

개별 비즈니스 서비스 안에서 Redis 클라이언트를 직접 `connect()`/`disconnect()`하지 않는다.
`PrismaService`와 동일한 패턴으로, NestJS DI로 주입되는 **단일 `RedisModule`**을 통해서만 접근한다.

**pub/sub 예외와 정리 의무**: 구독 모드 연결은 일반 명령을 못 쓰므로 `redis.duplicate()`로 전용 커넥션을 만드는 것이 정당하다.
다만 이 커넥션은 단일 `RedisModule` 밖에 있어 자동 정리되지 않으므로, 만든 서비스가 `OnModuleDestroy`에서 직접 `quit()`해 연결 누수를 막는다.

### 코드 리뷰/생성 시 응답 규칙

요청받은 코드가 클러스터 비호환(예: 해시태그 없는 멀티키 연산)이면, 바로 고치지 말고 **먼저 문제를 짚어 경고한 뒤** 수정본을 제시한다.

---
