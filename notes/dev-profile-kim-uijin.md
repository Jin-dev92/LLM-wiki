---
type: note
title: 개발자 프로필 — 김의진
summary: 5년 차 풀스택 개발자의 강점·주력 스택·대표 성과를 한눈에 본 프로필. 다른 프로젝트가 "내 배경/주력 기술"을 참조할 때 진입점.
created: 2026-06-19
updated: 2026-06-19
visibility: private
provenance: extracted
tags: [profile, career, fullstack, backend, infra]
---

## 핵심
백엔드~인프라를 직접 책임지는 풀스택 개발자(5년 차). 일하는 방식은 일관된 패턴 —
**느린 건 빠르게(성능 최적화), 없는 건 만들고(1인 개발·아키텍처 설계), 반복되는 건 자동화**.
주력은 NestJS·Spring Boot 백엔드 + React/Next.js + AWS 인프라.

## 상세
### 주력 스택
- **백엔드**: NestJS/Node.js/TypeScript, Spring Boot/Java — 둘 다 실무 운영 경험. [[rules/stacks/nestjs]], [[rules/stacks/java]]
- **프론트**: React, Next.js, React Query
- **DB/캐시**: PostgreSQL, MySQL, MongoDB, Redis, Aurora
- **인프라**: AWS(ECS/ECR/RDS/CloudFront/Lambda), Docker, CI/CD(GitHub Actions·CodePipeline)
- **AI 활용**: Claude Code + MCP(Sentry·Notion) 워크플로우 자동화, Gemini API 파이프라인

### 반복되는 강점 패턴 (성과로 검증됨)
- **성능 최적화**: API 응답 1.2s→0.08s(93%), 15s→2s(87%), 로딩 3.2s→0.8s(80%) — 병렬 처리·캐싱·CDN·Read Replica·빌드 최적화
- **0→1 / 1인 풀스택**: EV 충전 구독 서비스 요구사항~배포 전 과정 1인, Spring→Django 마이그레이션 주도
- **자동화**: PR 작성/리뷰 자동화, 환율 갱신 스케줄러, 번역 자동화(오류율 80%↓), CI/CD
- **신뢰성**: Event-Driven 변경이력(EntityChangeLog), JWT 블랙리스트, 테스트 커버리지 80%+, 실시간 모니터링

### 도메인 경험
e-commerce(글로벌), 헬스케어 CRM/전자차트, AI 서버(차트 요약·상품 추천), 구독 결제, 게임 UI(검은사막).

### 자격/학력
정보처리기사(2024.12), SQLD(2020.12), 한국폴리텍 I 대학 유비쿼터스통신과(2016 졸).

## 관련
- [[sources/resume/resume-fullstack-2026]] — 출처 이력서 원문
- [[rules/stacks/nestjs]] · [[rules/stacks/java]] — 주력 백엔드 스택 운영 규칙
- [[projects/estate-server]] — NestJS+Prisma 백엔드 (현재 작업 맥락)
