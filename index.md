---
type: moc
title: Index
summary: 위키의 모든 페이지를 한곳에 모은 마스터 카탈로그. /ingest 시 자동 갱신.
created: 2026-06-18
updated: 2026-06-20
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
- [[dev-profile-kim-uijin]] — 5년 차 풀스택 개발자 프로필(강점·주력 스택·대표 성과).
- [[resume-writing-principles]] — 경력직 이력서 작성 원칙(상단 임팩트·Before→After·문제→대처·기여도).
- [[agent-skill-authoring]] — SKILL.md 작성 패턴(description=무엇+Use when, 계층 로딩, rigid/flexible).
- [[agent-skill-archetypes]] — 에이전트 스킬 4가지 아키타입(워크플로우/취향/도구어댑터/프레임워크) + 도입 판단 규칙.
- [[ui-ux-reference]] — 토스·당근·Linear·Stripe 등 11개 서비스 UI/UX를 모방용 패턴·디자인 토큰·안티패턴으로 정리.

## sources
- [[sources/llm-agents/building-effective-agents]] — Anthropic 실용 에이전트 구축 가이드 (web).
- [[sources/llm-agents/andrew-ng-agentic-workflows-sequoia]] — Andrew Ng Sequoia 강연, 4패턴+실험 (youtube).
- [[sources/claude-code/claude-code-core-tools-guide]] — Claude Code 추천 도구 6종 설치 가이드 (local).
- [[sources/claude-code/claude-code-gstack-superpowers-guide]] — GStack+Superpowers 워크플로우 가이드 (local).
- [[sources/claude-code/agent-skills-repos-analysis]] — 공개 스킬 레포 4종 분석·도입 판정 (memo).
- [[sources/resume/resume-fullstack-2026]] — 김의진 이력서, 경력·성과·스택·자격증 (pdf).
- [[sources/career/resume-recruiter-guide]] — 채용담당자 관점 이력서 개선 가이드 (local).

## rules
- [[rules/company/git-pr]] — 팀 Git/PR + 공통 프로세스(문서위치·TDD·justfile) (company).
- [[rules/stacks/java]] — Java/Spring 백엔드 규칙(QueryDSL·DTO·DDD·Liquibase 등) (stack).
- [[rules/stacks/nestjs]] — NestJS 일반 규칙(모듈·검증·설정키·트랜잭션, Prisma 기준) (stack).
- [[rules/stacks/prisma]] — Prisma DB 규칙(cuid·논리삭제·마이그레이션) (stack).
- [[rules/stacks/nestjs-test]] — NestJS 테스트 코드 규칙 (stack).
- [[rules/stacks/frontend]] — React Query+Zustand+TS FE 개발 규칙 (stack).
- [[rules/stacks/react]] — React 함수형 컴포넌트 베이스 룰 (stack).
- [[rules/stacks/nextjs]] — Next.js App Router 규칙(Server/Client 경계·Server Action·NEXT_PUBLIC_ 민감키 금지) (stack).

## projects
- [[projects/estate-server]] — NestJS+Prisma+Kafka 플랫폼 백엔드 (최신 컨벤션 기준).
- [[projects/fanddle-server]] — Fanddle 플랫폼 백엔드 (Express+Sequelize 모노레포).

## guide
- [[guide/ai/claude-code-setup]] — Claude Code 환경 온보딩(플러그인·gstack 스킬·MCP·추천도구) + 동기화.
- `guide/ai/claude-global.md` — 글로벌 `~/.claude/CLAUDE.md` 사본(원문 보관).
- `guide/mcp/` — Serena · AWS MFA+MySQL MCP 설정 가이드(원문 보관).
- `guide/aws/` — RDS 스냅샷 → 로컬 MySQL 복구 가이드(원문 보관).
- `guide/github-actions/` — Claude PR 자동 리뷰 워크플로우 설정·요약(원문 보관).

## decisions (ADR)
- `docs/decisions/ADR-0001-keep-gstack-superpowers-over-agent-skills.md` — Claude Code 기반을 gstack+superpowers로 유지(agent-skills 전환 안 함) + 3종 cherry-pick. flip condition 포함.

## MOC
- [[MOC/Home]] — 위키 진입점.
