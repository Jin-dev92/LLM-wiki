---
type: note
title: Agentic Context Platform (통합 Context Provider)
summary: 팀 자산의 메타데이터를 기계가 읽는 형태로 자산화해, 사람과 AI 에이전트에게 동일한 최신 컨텍스트를 한 곳에서 공급하는 플랫폼. 사람의 탐색비용과 AI 환각을 같은 뿌리에서 해소.
created: 2026-06-24
updated: 2026-06-24
visibility: private
provenance: extracted
tags: [llm, agent, context, metadata, openmetadata, data-catalog, lineage, harness]
---

## 핵심
사람이 답을 못 찾는 것과 AI 에이전트가 환각을 내는 것은 **같은 뿌리 — Context의 부재**다(흩어져 있고 사람 머릿속에만 있어 기계가 못 읽음). 해법은 팀 자산의 메타데이터(description·lineage·profile·owner·quality)를 **기계가 읽을 수 있는 형태로 자산화**해, 사람(UI)과 에이전트(API/스킬)에게 **동일한 최신 컨텍스트를 한 곳에서** 공급하는 것이다. 이것이 "AI 도입의 인프라"로서의 Context Provider다.

## 상세
"두 개의 문, 같은 한 곳": 사람은 UI·SSO로, 에이전트는 스킬·API로 같은 컨텍스트 저장소에 접근한다. 이는 [[harness-engineering]]의 한 축 — 모델 외부에 "도메인 컨텍스트"를 인코딩해 에이전트가 헤매지 않게 하는 부품 — 의 데이터 플랫폼 버전이다.

### 설계 원칙 (NAVER 플레이스 사례)
1. **한 곳에서 질의** — 시스템마다 흩어진 메타를 한 곳에서 의미 있게.
2. **지식의 상향평준화** — 담당자만 알던 조회법을 누구나 같은 답으로.
3. **자동 수집이 전제** — 사람이 채우는 카탈로그는 개인 성실성에 의존해 stale. 메타데이터는 코드·시스템 변경과 함께 따라와야 한다. (그래서 "문서화(CLAUDE.md/위키)" 접근은 동적 속성 표현 불가·노후화로 탈락시킴.)
4. **팀 스택과 마찰 없는 연동** — 연동·운영 비용이 0에 가깝게.

### 구현 골격
- **파이프라인:** Sources(Hive·Iceberg·OpenSearch·REST·Argo) → Ingestion(스케줄러로 메타만 수집) → 카탈로그(OpenMetadata) → Serve → Interface(사람=UI, 에이전트=API·`SKILL.md`).
- **품질을 컨텍스트로:** 데이터 품질 검증(Deequ: 완전성·유일성·분포·참조무결성) 결과를 카탈로그에 누적 → 에이전트가 **신뢰 등급**과 함께 자산을 추천. → [[self-improvement-loop]]의 센서가 공급하는 신호와 같은 역할.
- **Column-level lineage:** 테이블 단위가 아니라 컬럼 사용까지 추적해 "바꾸면 어디가 깨지나"를 답한다.
- **에이전트 인터페이스(SKILL):** 에이전트가 코드 작성 *전에* `search_tables`/`get_table`/`get_lineage`로 스키마·계보를 스스로 조회 → [[agent-skill-authoring]]의 SKILL.md로 "도구의 문"을 연 형태.
- **Description 자동화:** LLM이 코드를 읽어 초안 → **사람은 작성자가 아니라 검수자(PR 리뷰)**. 이는 [[ai-experience-on-resume]]의 human approval gate, [[harness-engineering]]의 "사고는 위임해도 책임(taste)은 위임 못 한다"와 같은 사상.

### 메모
- provenance: 사례 사실(원칙·파이프라인·도구)은 발표 슬라이드에서 extracted. "harness-engineering의 데이터 플랫폼 버전", "self-improvement-loop 센서와 동치"라는 묶음은 내 해석(inferred).
- 대비점: 이 위키 자체는 "문서화" 접근([[harness-engineering]]이 위키를 하네스 부품으로 봄)인데, 본 사례는 그 방식을 stale로 탈락시켰다. 위키의 강점은 추론·내러티브 지식, 본 플랫폼의 강점은 자동수집되는 구조적/동적 메타 — **상호 보완**으로 읽는 게 맞다.

## 관련
- [[harness-engineering]] — Context Provider = 에이전트용 하네스(컨텍스트 공급 축)
- [[agent-skill-authoring]] — 에이전트에 자산 조회 도구를 여는 SKILL.md
- [[self-improvement-loop]] — 품질검증 결과를 신뢰 신호로 공급하는 센서
- [[ai-experience-on-resume]] — 작성자→검수자 전환 / human approval gate 사상 공유
- [[workflow-vs-agent]] — 에이전트가 동적으로 컨텍스트를 조회하는 자율성의 전제
- 출처: [[sources/llm-agents/context-provider-naver-d2]]
