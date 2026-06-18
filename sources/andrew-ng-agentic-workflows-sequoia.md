---
type: source
title: What's Next for AI Agentic Workflows — Andrew Ng at Sequoia AI Ascent
summary: Andrew Ng의 14분 강연. 4가지 Agentic Reasoning 패턴 소개 + 실험으로 효과 입증(GPT-3.5+에이전트 루프가 GPT-4 제로샷 능가, HumanEval).
created: 2026-06-18
updated: 2026-06-18
visibility: private
url: https://www.youtube.com/watch?v=sal78ACtGTc
author: Andrew Ng (DeepLearning.AI, AI Fund)
ingested_via: youtube
tags: [llm, agent, andrew-ng, sequoia, agentic-reasoning]
---

## 요약
Sequoia Capital AI Ascent에서 Andrew Ng이 발표한 약 14분 분량의 강연. AI 에이전트 워크플로가 기존 LLM 사용 방식을 근본적으로 바꾸며, 이것이 차세대 파운데이션 모델 출시만큼이나 중요한 트렌드라고 주장한다. 4가지 Agentic Reasoning 디자인 패턴(Reflection, Tool use, Planning, Multi-agent collaboration)을 소개하고, 실험 데이터로 에이전트 워크플로의 효과를 증명한다. 핵심 메시지: GPT-3.5에 에이전트 루프를 씌우면 GPT-4 제로샷을 능가할 수 있다(HumanEval 기준).

## 추출한 영구노트
- [[agentic-reasoning-design-patterns]]
- [[workflow-vs-agent]]

## 출처 원문 메모
- HumanEval 벤치마크: GPT-3.5 제로샷 48% → 에이전트 워크플로 74.4%, GPT-4 제로샷 67%
- Multi-agent Debate (Du et al., 2023): Biographies 66→73.8%, MMLU 63.9→71.1%, Chess 29.2→45.2%
- Reflection, Tool use는 robust(거의 항상 작동), Planning/Multi-agent는 emerging(가끔 놀라운 결과)
- "에이전트에게 작업을 위임하고 기다리는 패턴에 익숙해져야 한다"
