---
type: project
title: estate-server
summary: NestJS 11 + Prisma(PostgreSQL) + Kafka 기반 건물주/임차인 플랫폼 백엔드. 최신 팀 컨벤션의 기준 프로젝트(학습용). 워커(persistence/audit/notification/outbox) 분리.
created: 2026-06-18
updated: 2026-06-18
visibility: private
provenance: extracted
status: active
tags: [project, nestjs, prisma, kafka, backend]
source_file: estate-server (CLAUDE.md, package.json, prisma/, src/, docs/superpowers)
---

## 목표
건물주-임차인 플랫폼 백엔드. 인증·매물/초대·게시판·채팅·알림·감사로그를 마일스톤(M0~M10)으로
구축하며, 가장 최신 팀 개발 컨벤션(Prisma 기반)을 검증하는 기준 프로젝트(학습용).

## 스택 / 아키텍처
- **런타임**: NestJS 11 (Express), TypeScript
- **DB**: Prisma 6 + PostgreSQL ([[rules/stacks/prisma]])
- **메시징**: Kafka (kafkajs) — Outbox 패턴 + DLQ/백오프
- **워커 분리**: persistence / audit / notification / outbox (마이크로서비스)
- **인증**: JWT + Passport, 역할(OWNER/TENANT/ADMIN)
- **부가**: Redis(캐시·rate limit), Swagger, Sentry, 부하테스트(load/)
- **도메인 모듈**(`src/`): auth, property, board, chat, notification, audit, outbox, events, common, config, redis, prisma, workers

## 적용 규칙 (위키)
- [[rules/stacks/nestjs]] — NestJS 일반 규칙 (모듈/DTO/Swagger/설정키 상수화/트랜잭션)
- [[rules/stacks/prisma]] — Prisma DB 규칙 (cuid·논리삭제·마이그레이션)
- [[rules/stacks/nestjs-test]] — 테스트 코드 규칙
- [[rules/company/git-pr]] — 커밋/PR 컨벤션 (estate-server는 `[티켓명]{기능}: 한글` 변형 사용)

## 의사결정 로그
- 논리삭제: 물리삭제 → `deletedAt` 전환 (5개 엔티티), `@@index([deletedAt])` (2026-06-13 설계 확정)
- Outbox 패턴 + DLQ/백오프로 Kafka 발행 신뢰성 확보
- 커밋 컨벤션: `[티켓명]{기능}: {한글 설명}` (티켓 없으면 마일스톤 `M1` 등)

## 삽질/배운 것
- (이 프로젝트에서 얻은 교훈을 `notes/`로 추출하면 위키링크로 연결)
