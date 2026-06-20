---
type: note
title: 에이전트 스킬 4가지 아키타입
summary: "스킬"이라 불리는 것들은 사실 4종 — 워크플로우 묶음 / 도메인-취향 / 도구 어댑터 / 프레임워크 라이브러리. 도입 판단(중복·충돌) 시 같은 종류끼리 비교해야 한다.
created: 2026-06-20
updated: 2026-06-20
visibility: private
provenance: inferred
tags: [claude-code, agent-skills, skill-design]
---

## 핵심
공개 "스킬" 레포들을 비교하면 같은 단어 아래 **본질이 다른 4가지**가 섞여 있다.
도입을 검토할 땐 "스킬이냐"가 아니라 **어느 아키타입이냐**로 보고, 같은 종류끼리만 중복/충돌을 따져야 한다.

## 상세

| 아키타입 | 정체 | 예시 | 충돌·중복 판단 |
|---|---|---|---|
| **워크플로우 묶음** | "개발하는 법" 방법론(spec→build→ship, TDD) | addyosmani/agent-skills, superpowers | 같은 종류끼리 **명령어·방법론 충돌** 큼 → 택1 |
| **도메인-취향** | 특정 영역 품질/미감 규칙 | taste-skill(프론트 안티-슬롭) | 영역이 다르면 공존 OK, 같은 영역이면 layer 구분(정적 체크리스트 vs 런타임 QA) |
| **도구 어댑터** | 외부 실행 도구 + 사용법 지시 SKILL | graphify(코드→지식그래프), gbrain | 방법론 아님. 기능 중복(graphify↔gbrain)만 주의 |
| **프레임워크 라이브러리** | 원자적 프레임워크 다수 + 체이닝 명령 | pm-skills(68 skills/9 plugins) | 도메인 단위로 켜고 끔. 충돌 적음, context 비용 큼 |

### 함의 (도입 결정에 쓰는 규칙)
1. **워크플로우 묶음은 1개만.** superpowers(HOW)+gstack(실행)은 역할 분담이라 공존하지만, 거기에 addyosmani를 더하면 세 번째 "개발하는 법"이 충돌.
2. **도구 어댑터는 기능 중복만 확인**(graphify vs gbrain). 방법론과 안 겹치므로 별개 축.
3. **도메인-취향/프레임워크 라이브러리는 cherry-pick.** 통째 설치 대신 필요한 SKILL.md만(예: taste-skill의 redesign 1개).
4. **공통 비용**: 모든 스킬 description은 시스템 프롬프트에 로드됨 → 많이 깔수록 context 팽창.

## 관련
- [[agent-skill-authoring]] — 각 아키타입이 공유하는 SKILL.md 작성 규약
- [[agent-skills-repos-analysis]] — 종합 출처(4개 레포)
- [[workflow-vs-agent]] — "워크플로우 묶음" 아키타입이 인코딩하는 대상
