---
type: source
title: Redis Cluster 코딩 룰 (Claude 시스템 프롬프트)
summary: 사용자가 작성한 Claude Code용 NestJS+Redis Cluster 코딩 가이드라인 — CROSSSLOT 방지·TTL 필수·DI 연결관리.
created: 2026-07-05
updated: 2026-07-05
visibility: private
url:
author: 김의진 (사용자 작성)
ingested_via: pdf
tags: [backend, redis, nestjs, claude-code]
---

## 요약
NestJS+Redis Cluster 환경에서 Claude에게 코드 생성/리뷰를 시킬 때 지키게 할 규칙 3가지를 정리한 시스템 프롬프트 조각: (1) 멀티키 연산의 CROSSSLOT 방지용 해시태그, (2) SET/HSET TTL 필수 + mutation 시 캐시 무효화, (3) Redis 연결을 개별 서비스가 아닌 DI 싱글턴으로 관리. 클러스터 비호환 코드를 요청받으면 먼저 경고 후 수정본을 주라는 응답 형식 규칙도 포함.

## 추출한 영구노트
- [[redis-cluster-crossslot-prevention]]
- [[redis-mandatory-ttl-and-cache-invalidation]]
- [[redis-connection-management-via-di]]

## 출처 원문 메모
로컬 파일 `~/Downloads/cc-redis-rule.md`(사용자 작성, 2026-07-05 위키 반입). 원문은 [[rules/stacks/redis]]로 정제되어 팀 공유용 규칙(scope: stack)으로 편입됨.
