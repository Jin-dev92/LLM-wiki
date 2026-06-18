---
type: source
title: Building Effective Agents
summary: Anthropic의 실용 에이전트 구축 가이드. 복잡한 프레임워크 대신 단순·조합 가능한 패턴을 권고하고 Workflow/Agent를 구분.
created: 2026-06-18
updated: 2026-06-18
visibility: private
url: https://www.anthropic.com/engineering/building-effective-agents
author: Erik S., Barry Zhang (Anthropic)
ingested_via: web
tags: [llm, agent, workflow, anthropic]
---

## 요약
Anthropic 엔지니어링팀이 여러 산업의 팀과 협업하며 발견한 AI 에이전트 구축의 실용적 패턴 정리. 핵심 메시지는 "성공적인 구현은 복잡한 프레임워크가 아닌 단순하고 조합 가능한 패턴에서 나온다"는 것. Workflows(사전 정의 흐름)와 Agents(LLM이 흐름을 동적으로 제어)를 구분하고, 5가지 핵심 workflow 패턴(Prompt Chaining, Routing, Parallelization, Orchestrator-workers, Evaluator-optimizer)을 소개한다.

## 추출한 영구노트
- [[workflow-vs-agent]]
- [[agentic-workflow-patterns]]

## 출처 원문 메모
- 2024-12-19 발행
- 단순성, 투명성, ACI(Agent-Computer Interface) 정교화를 핵심 원칙으로 강조
- 실무 사례: 고객지원, 소프트웨어 개발(SWE-bench)
