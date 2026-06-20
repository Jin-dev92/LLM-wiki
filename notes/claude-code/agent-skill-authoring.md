---
type: note
title: 에이전트 스킬 작성 패턴 (SKILL.md anatomy)
summary: 좋은 SKILL.md의 frontmatter·본문 구조 규약 — name+description("무엇+Use when"), 짧은 본문 + 온디맨드 references, Rationalizations/Red Flags/Verification, rigid vs flexible.
created: 2026-06-20
updated: 2026-06-20
visibility: private
provenance: extracted
tags: [claude-code, agent-skills, skill-design]
---

## 핵심
스킬은 description으로 "발견"되고 본문으로 "실행"된다. 그래서 **frontmatter `description`에는
무엇(3인칭) + 언제(Use when 트리거)를 적되 프로세스 단계는 넣지 않는다** — 에이전트가 요약만
따르고 본문을 안 읽기 때문. 본문은 짧게 두고 무거운 명세는 `references/`로 분리해 필요할 때만 로드한다.

## 상세

### frontmatter 규약 (addyosmani skill-anatomy 기준)
- `name`: 소문자-하이픈, 디렉터리명과 일치.
- `description`: "무엇을 하는지(3인칭) + Use when 트리거", 최대 1024자. **프로세스 단계 금지.**
- `SKILL.md`만 필수. `scripts/`·보조 md는 실제로 쓸 때만 추가.

### 권장 본문 섹션 패턴
Overview / When to Use(+제외 조건) / Workflow(번호 단계) /
**Common Rationalizations 표**(스킵 핑계 ↔ 현실) / **Red Flags**(위반 징후) /
**Verification 체크리스트**(완료 기준·증거). superpowers 스킬들과 같은 DNA.

### 토큰 비용 평탄화 (계층 로딩)
- 본문은 핵심만, 무거운 명세(추출 스펙·쿼리·훅)는 `references/<...>.md`로 빼고 "필요할 때만 로드".
- graphify가 모범: `skill.md`(짧음) + `skills/<agent>/references/` + Fast-path 가드(이미 산출물 있으면 단계 스킵).
- → 이 위키의 **계층 검색**(제목/태그/summary 먼저, 본문 나중) 원칙과 동형.

### rigid vs flexible
- **rigid**(TDD·debugging·graphify): 단계·가드·Red Flag로 규율을 강제. 그대로 따른다.
- **flexible**(취향·패턴): 원칙을 맥락에 적응. taste-skill의 "다이얼(VARIANCE/MOTION/DENSITY) + 브리프 추론"처럼 **주관 영역도 파라미터+추론 규칙으로 명세화**하면 재현 가능해진다.

### 멀티 에이전트 호환
한 소스를 Claude/Gemini/Cursor/Codex로 배포하려면 에이전트별 변종 파일을 둔다
(graphify `skill-codex.md`·`skill-copilot.md` …, addyosmani `.claude/`+`.gemini/`).

## 관련
- [[agent-skill-archetypes]] — 이 작성 규약이 적용되는 스킬의 4가지 형태
- [[agent-skills-repos-analysis]] — 추출 출처(4개 레포 분석)
- [[workflow-vs-agent]] — 스킬은 대개 "워크플로우"를 인코딩(동적 Agent와 대비)
- [[claude-code-setup]] — 실제 설치된 스킬 환경 온보딩
