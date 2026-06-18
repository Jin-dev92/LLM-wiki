---
type: moc
title: Index
summary: 위키의 모든 페이지를 한곳에 모은 마스터 카탈로그. /ingest 시 자동 갱신.
created: 2026-06-18
updated: 2026-06-18
visibility: private
tags: [index]
---

# Index — 마스터 카탈로그

`/ingest` 가 생성/병합 시 이 파일을 갱신한다. 연관성 검색은 여기와 각 노트의
제목/태그/summary를 먼저 훑는다(계층 검색).

## notes
- [[workflow-vs-agent]] — Workflow는 미리 정의한 흐름, Agent는 LLM이 동적 결정. 트레이드오프.
- [[agentic-workflow-patterns]] — Anthropic의 재사용 5패턴(Chaining·Routing·Parallel·Orchestrator·Evaluator).
- [[agentic-reasoning-design-patterns]] — Andrew Ng의 4패턴(Reflection·Tool use·Planning·Multi-agent).

## sources
- [[sources/building-effective-agents]] — Anthropic 실용 에이전트 구축 가이드 (web).
- [[sources/andrew-ng-agentic-workflows-sequoia]] — Andrew Ng Sequoia 강연, 4패턴+실험 (youtube).

## rules
- [[rules/company/git-pr]] — 팀 Git/PR + 공통 프로세스(문서위치·TDD·justfile) (company).
- [[rules/stacks/java]] — Java/Spring 백엔드 규칙(QueryDSL·DTO·DDD·Liquibase 등) (stack).
- [[rules/stacks/nestjs]] — NestJS 일반 규칙(모듈·검증·트랜잭션) (stack).
- [[rules/stacks/nestjs-test]] — NestJS 테스트 코드 규칙 (stack).
- [[rules/stacks/frontend]] — React Query+Zustand+TS FE 개발 규칙 (stack).
- [[rules/stacks/react]] — React 함수형 컴포넌트 베이스 룰 (stack).

## MOC
- [[MOC/Home]] — 위키 진입점.
