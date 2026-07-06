---
type: note
title: Redis 캐시 TTL 필수 + 무효화 정책
summary: 모든 캐시 쓰기에는 명시적 TTL을 넣고, 원본 데이터가 변경되면 관련 캐시 키를 함께 삭제해 DB-캐시 불일치를 막는다.
created: 2026-07-05
updated: 2026-07-06
visibility: private
provenance: extracted
tags: [redis, cache, ttl, backend, counter]
---

## 핵심
`SET`/`HSET` 등으로 캐시를 쓸 때 만료 시간(TTL, 예: `EX 3600`)을 빼먹지 않는다. TTL 없는 캐시는 stale 데이터가 무한정 남아 DB와의 불일치를 키운다.

## 상세
TTL만으로는 부족하다 — Prisma/TypeORM 등으로 원본 데이터를 Create/Update/Delete하는 시점에, 그와 연결된 캐시 키를 함께 `DEL`(무효화)해야 한다. "쓰기 후 캐시 무효화"를 빠뜨리면 TTL이 만료되기 전까지 사용자가 오래된 값을 계속 보게 된다. Mutation 로직을 작성/리뷰할 때마다 "이 변경이 무효화해야 할 캐시 키가 있는가"를 체크리스트로 삼는다.

## capped 리스트도 캐시 쓰기다
`LPUSH` + `LTRIM`으로 "최근 N개"만 유지하는 리스트에도 TTL이 필요하다. `LTRIM`은 리스트 **길이**만 자를 뿐 키를 만료시키지 않으므로, `EXPIRE`를 함께 걸지 않으면 활동이 끊긴 키가 영구 잔존한다. push마다 `EXPIRE`를 다시 거는 슬라이딩 방식이 흔하다.

## 파생 카운터(derived counter) 패턴
미읽음 수·좋아요 수처럼 **DB 집계(`COUNT(*)`)의 파생 캐시**인 카운터는 TTL 없는 "영속 카운터"의 유혹에 빠지기 쉽다. 대신 진실 원천(DB) 우선 원칙으로 다음을 따르면 TTL 규칙을 지키면서도 캐시 유실이 영구화되지 않는다.

- **증감은 키가 존재할 때만**(Lua `EXISTS` 가드). 미스 키에 `INCR`하면 `0→1`로 실제 카운트를 잃으므로 손대지 않고, 재구축은 읽기 경로에 맡긴다.
- **읽기는 "카운터 우선 → 미스면 DB `COUNT` → `SET NX EX` 백필"**. `get`은 미스를 `null`로 신호해 값 `0`과 구분한다.
- **TTL은 백필만 소유**한다(증감 때는 갱신 안 함) → TTL이 drift(캐시-DB 불일치)의 상한이 된다.
- 리셋은 값 `0` 덮어쓰기보다 **키 삭제(`DEL`)**가 안전하다 — 다음 읽기가 DB로 재집계해 리셋-증감 경합을 자가 교정한다.

미스 재구축 중 새 증감이 끼면 최대 TTL만큼 ±1 drift가 남는 best-effort이며, TTL이 그 상한 역할을 한다.

## 관련
- [[redis-cluster-crossslot-prevention]]
- [[redis-connection-management-via-di]]
- [[rules/stacks/redis]]
- [[rules/stacks/prisma]]
