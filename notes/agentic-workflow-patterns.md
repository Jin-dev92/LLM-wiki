---
type: note
title: Agentic Workflow 5가지 패턴
created: 2026-06-18
updated: 2026-06-18
visibility: private
tags: [llm, agent, workflow, pattern]
---

## 핵심
LLM 기반 시스템에는 재사용 가능한 5가지 워크플로 패턴이 있다. 단순한 것부터 조합하는 전략이 복잡한 프레임워크보다 실제 운영에서 더 안정적이다.

## 상세
1. **Prompt Chaining**: 큰 작업을 순서가 있는 작은 단계로 쪼개 순차 처리. 중간 결과를 다음 프롬프트의 입력으로 연결.
2. **Routing**: 입력을 분류해 전문화된 처리 경로로 분기. 예: 고객 문의 종류별 담당 LLM 연결.
3. **Parallelization**: 독립적인 하위 작업을 동시에 처리하거나, 동일 작업에 여러 LLM 응답을 받아 투표(Voting).
4. **Orchestrator-Workers**: 중앙 LLM이 계획 수립·작업 분배를 담당하고, Worker LLM들이 실행 후 결과를 보고.
5. **Evaluator-Optimizer**: 생성 LLM과 평가 LLM을 분리해 반복 개선 루프를 구성.

## 관련
- [[workflow-vs-agent]]
- 출처: [[sources/building-effective-agents]]
