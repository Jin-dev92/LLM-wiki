---
type: note
title: Redis 캐시 TTL 필수 + 무효화 정책
summary: 모든 캐시 쓰기에는 명시적 TTL을 넣고, 원본 데이터가 변경되면 관련 캐시 키를 함께 삭제해 DB-캐시 불일치를 막는다.
created: 2026-07-05
updated: 2026-07-05
visibility: private
provenance: extracted
tags: [redis, cache, ttl, backend]
---

## 핵심
`SET`/`HSET` 등으로 캐시를 쓸 때 만료 시간(TTL, 예: `EX 3600`)을 빼먹지 않는다. TTL 없는 캐시는 stale 데이터가 무한정 남아 DB와의 불일치를 키운다.

## 상세
TTL만으로는 부족하다 — Prisma/TypeORM 등으로 원본 데이터를 Create/Update/Delete하는 시점에, 그와 연결된 캐시 키를 함께 `DEL`(무효화)해야 한다. "쓰기 후 캐시 무효화"를 빠뜨리면 TTL이 만료되기 전까지 사용자가 오래된 값을 계속 보게 된다. Mutation 로직을 작성/리뷰할 때마다 "이 변경이 무효화해야 할 캐시 키가 있는가"를 체크리스트로 삼는다.

## 관련
- [[redis-cluster-crossslot-prevention]]
- [[redis-connection-management-via-di]]
- [[rules/stacks/redis]]
- [[rules/stacks/prisma]]
