---
type: note
title: 자가개선 루프 (Self-improvement Loop)
summary: 에이전트가 가이드(방향)와 센서(관찰)를 받아 스스로 작업을 검증·교정하며 반복하는 회로. Evaluator-Optimizer 패턴의 평가자를 느린 LLM 대신 결정론적 검증(테스트)으로 대체한 형태.
created: 2026-06-22
updated: 2026-06-22
visibility: private
provenance: extracted
tags: [llm, agent, loop, verification, pattern]
---

## 핵심
**자가개선 루프**는 에이전트가 한 번에 작업을 끝내지 못해도, 틀리면 스스로 고쳐 다시 돌릴 수 있게 하는 회로다. "에이전트로 어디까지 일을 맡길 수 있는가"는 결국 *이 루프를 닫을 수 있느냐*로 갈린다. 에이전트 시대 사람의 일은 이 회로를 닫는 것.

## 상세
- **두 부품**:
  - **가이드(Guide)** — 행동하기 *전에* 방향을 잡아줌: `AGENTS.md`, MCP, 스킬, 룰, LSP. (명세/규칙/연산을 명료히 제공해 에이전트가 헤매지 않게)
  - **센서(Sensor)** — 행동한 *후에* 관찰해 자가교정을 도움: 테스트·린터·타입체커·AI 코드리뷰.
- **하나가 둘을 겸할 수 있다**: E2E 테스트 코드는 *돌리면* 센서, *읽으면* 명세(가이드)다 → [[spec-as-code]]. 한 벌만 짜면 가이드+센서가 동시에 해결된다.
- **Evaluator-Optimizer와의 관계**: Anthropic "Building Effective Agents"의 [[agentic-workflow-patterns]] 중 Evaluator-Optimizer(생성 LLM ↔ 평가 LLM 분리 후 피드백 반복)와 동형이다. 단, 채점자가 LLM이면 비싸고 비결정적이고 느리다 → 그 자리에 **결정론적·빠른·저렴한 검증(Playwright 등)**을 넣는 것이 실전 핵심.
- **CI로 회로 닫기**: 에이전트가 PR을 올리면 → CI에서 테스트 실행 → 실패 시 머지 차단 + 결과 코멘트 + 실패 컨텍스트(trace) 아티팩트 → 에이전트가 이를 받아 디버깅·수정·재반영. 사람이 PR/CI 로그로 일하는 방식과 동일.
- 이 루프는 [[harness-engineering]]가 만들어내는 결과물이다.

## 관련
- [[harness-engineering]] — 루프를 떠받치는 환경
- [[agentic-workflow-patterns]] — Evaluator-Optimizer 패턴
- [[spec-as-code]] — 가이드이자 센서인 테스트
- [[playwright-e2e-for-agents]] — 결정론적 센서의 구현
- [[workflow-vs-agent]]
- 출처: [[sources/llm-agents/playwright-e2e-harness-naverpay]]
