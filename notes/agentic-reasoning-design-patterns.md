---
type: note
title: Agentic Reasoning 4가지 디자인 패턴 (Andrew Ng)
summary: Andrew Ng의 4패턴 — Reflection·Tool use는 성숙(거의 항상 작동), Planning·Multi-agent는 이머징(강력하나 불안정).
created: 2026-06-18
updated: 2026-06-18
visibility: private
provenance: extracted
tags: [llm, agent, agentic-reasoning, pattern, andrew-ng]
---

## 핵심
AI 에이전트 시스템에는 성숙도가 다른 4가지 디자인 패턴이 있다. Reflection과 Tool use는 이미 실용 단계이고, Planning과 Multi-agent는 강력하지만 아직 불안정하다.

## 상세
Andrew Ng이 Sequoia AI Ascent에서 제시한 분류:

**성숙(Robust) — 거의 항상 작동:**
1. **Reflection**: LLM이 자신의 출력을 스스로 검토·비판·개선. 코드 생성 후 동일 LLM에 "이 코드의 버그를 찾아라"고 재프롬프팅하면 스스로 수정. 유닛 테스트 실패 → 원인 분석 → 재시도 루프도 Reflection의 응용.
2. **Tool use**: 웹 검색, 코드 실행, 이미지 생성/캡셔닝 등 외부 도구 연동. 분석(Wolfram Alpha, Code Interpreter), 리서치(검색/위키), 생산성(이메일/캘린더), 이미지(DALL-E) 등으로 카테고리화.

**이머징(Emerging) — 강력하지만 불안정:**
3. **Planning**: LLM이 작업을 단계별 계획으로 분해하고 실행. 아직 일관성이 낮음.
4. **Multi-agent collaboration**: 여러 에이전트가 역할 분담. Coder + Critic 패턴, ChatDev(소프트웨어 개발 시뮬레이션). Multi-agent Debate 실험에서 단일 에이전트 대비 유의미한 성능 향상 확인.

## 관련
- [[workflow-vs-agent]] — 에이전트와 워크플로의 근본 차이
- [[agentic-workflow-patterns]] — Anthropic의 5가지 워크플로 패턴 분류(유사 개념)
- 출처: [[sources/andrew-ng-agentic-workflows-sequoia]]
