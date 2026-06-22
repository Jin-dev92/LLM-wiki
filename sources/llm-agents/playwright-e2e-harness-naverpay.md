---
type: source
title: AI 에이전트를 위한 Playwright E2E 테스트 하네스 구축하기 (네이버페이 FE 발표)
summary: 네이버페이 대출 서비스 FE 팀이 AI 에이전트에게 "자기 작업을 검증할 결정론적 수단"으로 Playwright E2E 테스트를 도입한 사례. 하네스 엔지니어링·자가개선 루프·flaky 예방·Planner/Generator/Healer 에이전트를 다룸.
created: 2026-06-22
updated: 2026-06-22
visibility: private
url: https://www.youtube.com/watch?v=wo0Rsh9hlTo
author: 네이버페이(네이버 파이낸셜) 금융 FE (발표자명 불명확 — 검토 필요)
ingested_via: youtube
tags: [llm, agent, e2e, playwright, testing, harness]
---

## 요약
"AI가 쓴 코드 누가 검증하나"라는 문제의식에서 출발하는 ~32분 컨퍼런스 발표. AI 도입 후 PR은 1.5배(227→348) 늘었지만 검증이 병목이 됐고, 모델은 자기 코드에 관대해 컴포넌트 경계 결함을 놓친다. 해법으로 **모델 가중치 외부의 모든 것(=하네스)** 중 E2E 테스트를 택해, Playwright로 핵심 사용자 플로우(Critical User Flows)만 검증한다. 테스트 코드는 "읽으면 명세·돌리면 검증"인 두 얼굴이고, 풍부한 Trace가 에이전트의 디버깅 입력이 된다. Playwright 공식 Planner/Generator/Healer 에이전트 + 팀 자체 하네스(AGENTS.md·e2e-testing.md·유틸·MCP)로 작성·수리를 자동화하고, PR→CI→Healer로 **자가개선 루프**를 닫는다. 사람은 루프 설계·맥락 제공·하네스 진화를 담당.

## 추출한 영구노트
- [[harness-engineering]]
- [[self-improvement-loop]]
- [[spec-as-code]]
- [[playwright-e2e-for-agents]]

## 출처 원문 메모
- **문제 제기 (00:28~03:10)**: 에이전트가 Next.js 업그레이드 PR을 "완료"라고 했으나 뒤로가기 동작이 깨짐(확인 없이 종료). 보리스 체르니: "에이전트에게 검증 수단을 줘라 — 결과를 2~3배 끌어낸다."
- **하네스 정의 (01:10~01:35)**: 미첼 하시모토 — "실수가 다시 반복되지 않게 하는 것" = 모델 가중치 외부 모든 것(AGENTS.md·MCP·스킬·LLM 위키·테스트). LLM 위키가 하네스 구성요소로 명시됨.
- **컴포넌트 경계 결함 (04:24~05:06)**: unit(vitest+RTL)은 각각 통과해도 실제 브라우저에서 데이터 형식·상태·환경별 API 실패. AI 오류 대부분이 여기서 발생.
- **Testing Trophy (05:06~06:26)**: Kent C. Dodds. 비어있던 E2E 자리를 AI 시대엔 채워야(작성부담↓·검증중요도↑). 결론: Playwright E2E.
- **Spec-as-Code (07:17~08:00)**: `getByRole('button',{name:'한도조회'}).click()` 등은 곧 사용자 명세. 코드 바뀌면 명세도 같이.
- **Trace 리포트 (08:09~10:08)**: 스크린샷 타임라인·액션 before/after·DOM 스냅샷·콘솔·네트워크 = 구조화 데이터 → 에이전트가 원인분석·수정.
- **무엇을 테스트 (10:31~11:58)**: 6년차 17만줄 220+페이지. 전부는 비현실적 → Critical User Flows(실패 시 매출·데이터·신뢰 손상). 37개 라우트·CI 29연속·flaky 0.
- **작성 자동화 (12:07~15:45)**: codegen으로 생성→에이전트가 컨벤션화. Playwright 공식 3 에이전트 `npx playwright init-agents --loop=claude`: Planner(계획서)·Generator(코드)·Healer(수리). 분리 이유=한 LLM에 다 시키면 완성도↓.
- **하네스 4종 (16:41~17:32)**: ①AGENTS.md ②e2e-testing.md(단일출처) ③유틸/헬퍼(getAuthState·prefill) ④MCP(기획·디자인·이슈·PR).
- **팀 결정 3 (19:44~22:53)**: 테스트 독립성→API로 상태 prefill; 외부의존성 모킹(page.route / 서버는 env 분기); flaky 예방(하드대기 금지·시멘틱 셀렉터·burn-in `--repeat-each`).
- **CI 루프 (25:04~26:21)**: PR→CI e2e→실패 시 머지차단+PR코멘트+trace.zip 아티팩트→e2e-debug 스킬로 다운로드·분석→Healer 수정.
- **자가개선 루프 (26:21~28:29)**: 가이드(AGENTS.md·MCP·스킬)+센서(테스트·린터·타입체커). Playwright는 둘 다. Anthropic Evaluator-Optimizer와 동형이되 평가자를 결정론적 Playwright로 대체.
- **사람의 역할 (29:01~끝)**: 카파시 "사고는 위임해도 책임(taste)은 위임 못 함." ①루프 설계 ②도구·맥락 제공 ③모델 진화 시 하네스도 진화. Vercel: "올바른 행동을 쉽게 하도록 환경을 만들어라."
- 분석 방식: watch 스킬로 프레임 80장 + Groq Whisper 트랜스크립트(402세그먼트) 추출.
