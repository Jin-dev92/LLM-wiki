---
type: source
title: 사람과 AI Agent를 위한 통합 Context Provider 구축 (NAVER ENGINEERING DAY 2026)
summary: 플레이스 추천팀이 300+ 데이터 자산의 메타데이터를 OpenMetadata로 자동 수집·자산화해, 사람(UI/SSO)과 AI Agent(API/SKILL.md)에게 동일한 최신 컨텍스트를 공급하는 플랫폼 구축기.
created: 2026-06-24
updated: 2026-06-24
visibility: private
url: https://www.youtube.com/watch?v=0VdAZCYBwSU
author: 신동결·심효진 (NAVER 플레이스 AI 추천팀)
ingested_via: youtube
tags: [llm, agent, context, metadata, openmetadata, data-catalog, lineage, harness, naver]
---

## 요약
NAVER ENGINEERING DAY 2026 발표(13:26). 캡션·Whisper 미확보 → 슬라이드 25장(80프레임) 분석으로 재구성. 라이브 데모(Part 3)는 프레임에 거의 안 잡힘.

**핵심 주장:** "Context Provider는 AI 도입의 인프라다." 팀 데이터 자산의 메타데이터를 기계가 읽을 수 있는 형태로 자산화하면, 사람의 "물어볼 사람 찾기" 비용과 AI의 환각이 같은 뿌리(Context 부재)에서 동시에 풀린다.

### 문제 (Part 1)
- 플레이스 추천팀 자산 300+개(Hive 94테이블·Iceberg 32·OpenSearch 60인덱스·REST 14·Argo 122파이프라인).
- 매일 같은 4질문: ①이 테이블 뭐예요(description) ②언제 업데이트(freshness/lineage) ③데이터 몇 개(profile) ④삭제하려는데 쓰는 사람(downstream lineage/owner).
- 사람도 못 찾고(위키 stale·메신저 대기·담당자 흐름끊김·코드 난해), AI는 더 못함(team context 미학습 → 일반론·환각). 근본 원인 = **Context의 부재**(사람 머릿속에만, 기계가 못 읽음).

### 해법 (Part 2)
- 4원칙: ①한 곳에서 질의 ②지식 상향평준화 ③자동 수집 전제(개인 성실성 의존 X) ④팀 스택 마찰 없는 연동.
- 도구 비교: A)문서화(CLAUDE.md·사내 위키)=stale·동적속성 불가 / B)자체 카탈로그=운영부담·폐기경험 / **C)OpenMetadata 채택**=연동만 하면 lineage·owner·description·profile 자동, API+UI 지원.
- 개념: "두 개의 문, 같은 한 곳" — 사람(UI·SSO)·에이전트(SKILL·API)가 같은 Context Provider 접근.

### 구축 (Part 4)
- 5단계 단일 파이프라인: Sources → Ingestion(Argo Cron, 매일 새벽 03–05시 KST, 메타만) → OpenMetadata → Serve → Interface(사람=OM UI, 에이전트=REST API·SKILL.md). 사람이 등록 안 해도 다음날 자동 노출.
- 품질: **Deequ** 4체크(완전성·유일성·분포·참조무결성) → 결과를 OM에 누적 → 에이전트가 신뢰 등급과 함께 추천.
- Lineage: **column-level** 계보 그래프로 "누가 쓰나/바꾸면 어디 깨지나" 즉답.
- SKILL: openmetadata **SKILL.md**로 에이전트가 코드 작성 *전에* `search_tables`/`get_table`/`get_lineage`로 스키마·계보 조회.
- Description 자동화: 테이블 발견 → LLM 초안(SKILL이 코드 읽음) → **사람 PR 검수** → OM 반영. 사람 역할 **작성자→검수자**, 테이블당 수십분→수분(POC Hive 22/37).

### 임팩트·로드맵 (Part 5)
- Milvus(Vector DB) 연동, column-level/API lineage 강화, DQ를 drift·anomaly 자동알림, RBAC 거버넌스.

## 추출한 영구노트
- [[agentic-context-platform]]

## 출처 원문 메모
- 슬라이드 25장 기반(음성 transcript 미확보). 발표자 이메일은 사내 도메인(@navercorp.com) — 기록 생략.
- 흥미점: "문서화(CLAUDE.md/위키)"를 stale·동적속성 불가로 명시 탈락시키고 자동수집 메타로 감.
