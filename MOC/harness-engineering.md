---
type: moc
title: 하네스 엔지니어링 (Harness Engineering)
summary: AI 에이전트가 스스로 일하고 검증하도록 환경을 설계하는 주제 허브 — 하네스·자가개선 루프·검증 센서(E2E)·관련 패턴/룰을 모음.
created: 2026-06-22
updated: 2026-06-22
visibility: private
tags: [moc, llm, agent, harness]
---

# 하네스 엔지니어링 MOC

> "모델을 더 똑똑하게"가 아니라 "모델이 일할 환경을 설계"하는 주제. 에이전트가
> 스스로 작업을 검증·교정하는 [[self-improvement-loop]]를 닫는 것이 목표.

## 핵심 개념
- [[harness-engineering]] — 모델 가중치 외부의 모든 것. 진화하는 시스템.
- [[self-improvement-loop]] — 가이드 + 센서로 닫는 회로.
- [[spec-as-code]] — 테스트 = 읽으면 명세, 돌리면 검증.

## 검증 센서 구현
- [[playwright-e2e-for-agents]] — Playwright E2E를 결정론적 센서로 쓰는 실전 기법.

## 관련 패턴
- [[agentic-workflow-patterns]] — Evaluator-Optimizer 등 5패턴.
- [[workflow-vs-agent]] — Workflow vs Agent.
- [[agent-skill-authoring]] / [[agent-skill-archetypes]] — 하네스의 "도구(스킬)" 축.

## 적용 룰 (rules/)
- [[rules/stacks/frontend]] — 컴포넌트 경계 결함 예방 FE 룰.
- [[rules/stacks/nextjs]] — Next.js App Router 룰.
- [[rules/company/git-pr]] — CI/PR 검증 루프.

## 출처
- [[sources/llm-agents/playwright-e2e-harness-naverpay]] — 네이버페이 FE 발표.
- [[sources/llm-agents/building-effective-agents]] — Anthropic.
