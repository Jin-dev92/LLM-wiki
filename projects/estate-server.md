---
type: project
title: estate-server
summary: NestJS 11 + Prisma(PostgreSQL) + Kafka 기반 건물주/임차인 플랫폼 백엔드. 최신 팀 컨벤션의 기준 프로젝트(학습용). 워커(persistence/audit/notification/outbox) 분리.
created: 2026-06-18
updated: 2026-06-24
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

## Context Provider 적용 검토 (2026-06-24, provenance: inferred)
[[agentic-context-platform]](NAVER D2 발표) 개념을 estate-server에 맞게 줄여본 분석. **결론: 풀 플랫폼(OpenMetadata·수집 파이프라인·Deequ·data lineage)은 불필요**(estate는 이종 데이터자산 300+ 환경이 아님). 다만 발표의 핵심 원리 — *"에이전트가 코드 짜기 전에, 코드에서 자동 파생되는 최신 자산 정보를 스스로 조회"* — 는 작게 적용 가치가 있다. estate-server의 "자산" = Prisma 모델·Kafka 토픽·REST 엔드포인트·도메인 모듈/워커.

### 적용 후보 (가치/노력 순)
1. **스키마-우선 에이전트 규칙 (저노력·중간가치)** — 도메인 작업 전 `prisma/schema.prisma`(단일 진실원)와 해당 migration을 먼저 읽게 한다. fanddle(176모델)보다 통증은 작지만, 환각 필드·관계 오작성 예방. → [[agent-skill-authoring]]의 SKILL.md / [[rules/stacks/prisma]]와 결합.
2. **코드 파생 사실은 CLAUDE.md에 복붙 금지, 가리키기** — Swagger/OpenAPI(엔드포인트 카탈로그)와 Prisma schema는 이미 자동 생성되는 에이전트용 컨텍스트. CLAUDE.md에는 **손으로 쓴 컨벤션만**, 스키마·엔드포인트 같은 코드 파생 사실은 "여기를 보라"로 포인터만. (발표가 "문서화=stale"로 탈락시킨 교훈의 안전한 수용선 — [[agentic-context-platform]] 상호보완 관점.)
3. **Kafka 이벤트/토픽 카탈로그 (estate 고유·높은가치)** — Outbox 패턴 + DLQ/백오프 + events 모듈. 에이전트가 producer/consumer를 짤 때 **토픽명·페이로드 스키마·DLQ 동작**을 모르면 위험. events/outbox 모듈을 단일 참조점으로 정리(또는 가리키기)하면 "한 곳에서 질의"의 메시징 자산 버전. Kafka는 에이전트가 특히 틀리기 쉬워 ROI 높음.
4. **description 자동화 (작성자→검수자)** — Prisma 필드 주석 + Swagger `@ApiProperty` description을 LLM 초안 → 사람 PR 검수로 채움. 바운디드 작업. → [[ai-experience-on-resume]] human approval gate 사상.
5. **(보류) lineage/품질 신호** — column-level data lineage·Deequ는 불필요. 앱 버전 대체물은 "이 이벤트 누가 구독/이 모듈 누가 의존"(NestJS 모듈 그래프)·테스트 커버리지·Swagger를 신뢰 신호로. 우선순위 낮음.

### 권장 시작점
estate는 **최신 컨벤션 기준(학습)** 프로젝트이므로, ②(코드 파생 사실 포인터화) + ③(Kafka 이벤트 카탈로그)를 여기서 파일럿한 뒤 fanddle로 이식하는 흐름이 자연스럽다.

> ⚠️ 위 항목은 위키 프로젝트 노트 기반 분석이다. 실제 적용은 estate-server **repo의 `prisma/schema.prisma`·`src/events`·`src/outbox`·Swagger 설정을 직접 확인**한 뒤 확정할 것.

## 삽질/배운 것
- (이 프로젝트에서 얻은 교훈을 `notes/`로 추출하면 위키링크로 연결)
