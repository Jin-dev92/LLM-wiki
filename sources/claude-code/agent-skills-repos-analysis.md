---
type: source
title: 에이전트 스킬 레포 4종 분석 (agent-skills / taste-skill / graphify / pm-skills)
summary: addyosmani/agent-skills, Leonxlnx/taste-skill, safishamsi/graphify, phuryn/pm-skills 4개 공개 스킬 레포를 클론해 구조·도메인·설계패턴을 비교한 분석. 도입 여부 판정 포함.
created: 2026-06-20
updated: 2026-06-20
visibility: private
url:
author: 본인 (4개 GitHub repo 클론 후 직접 분석)
ingested_via: memo
tags: [claude-code, agent-skills, skill-design, gstack, superpowers, graphify]
---

## 요약
공개 "스킬" 레포 4개를 `/tmp/skills-analysis/`에 클론해 SKILL.md·README·구성요소를 비교했다.
핵심 결론: **4개는 같은 "스킬"이라는 단어를 쓰지만 사실 3가지 다른 종류**다 → [[agent-skill-archetypes]].
작성 규율의 정수는 [[agent-skill-authoring]]로 추출.

## 추출한 영구노트
- [[agent-skill-authoring]] — SKILL.md anatomy / 작성 패턴
- [[agent-skill-archetypes]] — 스킬 4가지 아키타입(워크플로우/취향/도구어댑터/프레임워크라이브러리)

## 레포별 분석 (원천 기록)

| 레포 | 종류 | 도메인 | 규모 | 패키징 |
|---|---|---|---|---|
| addyosmani/agent-skills | 워크플로우 스킬 묶음 | SW 엔지니어링 전 과정 | 24 skills + 8 commands + 4 agents + 6 hooks | Claude 플러그인 + Gemini/Cursor/Copilot 멀티호환 |
| Leonxlnx/taste-skill | 도메인 전문 스킬 묶음 | 프론트엔드 디자인(안티-슬롭) | 13 skills | `npx skills add` (vercel-labs 표준) |
| safishamsi/graphify | 실행 도구 + 스킬 어댑터 | 코드/문서 → 지식그래프 | 1 메가 스킬 × 13 에이전트 변종 | PyPI `graphifyy` + `graphify install` |
| phuryn/pm-skills | 프레임워크 스킬 마켓플레이스 | 제품관리(PM) | 68 skills + 42 chained commands, 9 플러그인 | 마켓플레이스(Claude/Codex/Cowork) |

### 도입 판정 (현재 환경 기준 — 플러그인 4 + gstack ~55 + superpowers)
- **addyosmani**: ❌ 설치 금지. `/spec /review /ship /test /build`가 gstack·superpowers와 **정면 충돌**. 단 `docs/skill-anatomy.md`는 참고가치(→ [[agent-skill-authoring]]).
- **taste-skill**: ⚠️ `redesign-existing-projects` 한정 가치 있음. 랜딩 전용 아님 — **기존 UI(CRM·대시보드 포함) 130+항목 정적 감사 체크리스트**. gstack `design-review`(런타임 시각 QA)와 layer 다름(상호보완). 통째 설치보다 SKILL.md 1개 복사 권장.
- **graphify**: ⚠️ 이 위키 철학과 가장 정렬(EXTRACTED/INFERRED/AMBIGUOUS = 우리 provenance). 단 Python/uv 필요·토큰비용·gstack `gbrain`과 기능 중복. **온보딩 문서에만 기재, 미설치**(→ [[claude-code-setup]] Section 5).
- **pm-skills**: ❌ 거의 불필요. PM 프레임워크는 도메인 불일치(`review-resume`만 눈에 띄나 이미 자체 이력서 워크플로우 보유).

## gstack+superpowers vs agent-skills (사용자 고민 메모, 2026-06-20)
- 셋 다 "개발하는 법" 방법론 → **동시 사용 시 명령어 충돌·context 팽창·상충 의견**.
- superpowers = HOW(방법론: brainstorming·TDD·debugging·plans), gstack = 무엇/실행(browse·qa·ship·design·investigate). 둘은 역할 분담이라 공존 OK.
- agent-skills는 이 둘의 영역을 **재중복** → 셋을 동시에 깔 이유 없음. 택1 구도.
- **결정(2026-06-20)**: 전환 안 함 + agent-skills 3종 cherry-pick(ADR·doubt·source). 근거·flip condition은 `docs/decisions/ADR-0001-keep-gstack-superpowers-over-agent-skills.md` 참조.

## 비고
- 클론 위치: `/tmp/skills-analysis/{addyosmani,taste-skill,graphify,pm-skills}` (임시).
- 정량 출처: `find -name SKILL.md` 카운트, 각 README/plugin.json, skill-anatomy.md.
