---
type: note
title: Workflow vs Agent 구분
created: 2026-06-18
updated: 2026-06-18
visibility: private
tags: [llm, agent, workflow]
---

## 핵심
Workflow는 코드로 미리 정의한 흐름에 따라 LLM을 호출하는 방식이고, Agent는 LLM 자신이 어떤 도구를 어느 순서로 쓸지 동적으로 결정하는 방식이다. 복잡성과 예측 가능성 사이의 트레이드오프가 핵심이다.

## 상세
- **Workflow**: 개발자가 흐름을 설계, LLM은 각 단계의 실행자 역할. 결과가 예측 가능하고 디버깅이 쉽다.
- **Agent**: LLM이 계획·실행·수정을 자율적으로 담당. 유연하지만 비용과 오류 전파 위험이 크다.
- Anthropic 권고: 우선 Workflow로 충분한지 확인 후 Agent로 격상. 필요 이상의 복잡도는 금물.
- "Agents are better for open-ended problems; workflows for well-scoped tasks."

## 관련
- [[agentic-workflow-patterns]]
- [[building-effective-agents]] — 출처: [[sources/building-effective-agents]]
