---
type: note
title: Redis Cluster CROSSSLOT 방지 (해시태그)
summary: Redis Cluster에서 멀티키 연산은 관련 키를 해시태그({})로 묶어 같은 슬롯에 둬야 CROSSSLOT 에러를 피한다.
created: 2026-07-05
updated: 2026-07-05
visibility: private
provenance: extracted
tags: [redis, cluster, cache, backend]
---

## 핵심
Redis Cluster는 키를 해시 슬롯 단위로 여러 노드에 분산 저장한다. `MGET`/`MSET`/멀티키 `DEL`/파이프라인처럼 한 번에 여러 키를 다루는 연산은, 대상 키들이 서로 다른 슬롯(노드)에 있으면 `CROSSSLOT` 에러로 실패한다.

## 상세
연관된 키들의 공통 부분을 `{}`로 감싸면, Redis는 그 중괄호 안의 문자열만 해싱해 슬롯을 결정한다. 따라서 같은 엔티티에 속한 키들(`{user:100}:profile`, `{user:100}:cart`)은 항상 같은 슬롯에 배치되어 멀티키 연산이 안전해진다.

```ts
// BAD — 두 키가 다른 슬롯에 흩어질 수 있음
redis.mget('user:100:profile', 'user:100:cart')

// GOOD — {user:100} 부분만 해싱되어 같은 슬롯 보장
redis.mget('{user:100}:profile', '{user:100}:cart')
```

싱글 인스턴스 Redis에서는 문제없이 동작하던 코드가 클러스터로 전환하는 순간 깨지는 대표적인 원인이다. 키 네이밍을 설계하는 단계부터 "이 키들이 함께 조회/조작될 일이 있는가"를 기준으로 해시태그 그룹을 미리 정해야 한다.

## 관련
- [[redis-mandatory-ttl-and-cache-invalidation]]
- [[redis-connection-management-via-di]]
- [[rules/stacks/redis]]
