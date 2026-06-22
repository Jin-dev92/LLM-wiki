---
type: note
title: Playwright E2E를 에이전트 검증 센서로 쓰기 (실전)
summary: AI 에이전트 시대의 Playwright 운영 실전 — Critical User Flows만 고르고, flaky를 작성 단계에서 잡고, 외부 의존성은 모킹, 상태는 API로 prefill, Trace는 에이전트 디버깅 입력으로, codegen+Planner/Generator/Healer로 작성·수리를 자동화.
created: 2026-06-22
updated: 2026-06-22
visibility: private
provenance: extracted
tags: [playwright, e2e, testing, agent, flaky, ci]
---

## 핵심
Playwright E2E 테스트는 실제 브라우저에서 사용자 경험을 검증하므로, AI가 만든 **컴포넌트 경계 결함**(unit은 통과하나 실제 실행에서 깨짐)을 잡는 결정론적 센서가 된다. 단, "잘 쓰려면" 범위 선정·flaky 예방·모킹·자동화 운영 원칙이 필요하다.

## 상세

### 무엇을 테스트할까 — Critical User Flows
- 대규모 코드베이스(예: 17만 줄·220+페이지)에서 전부 테스트는 비현실적. **실패하면 매출이 막히거나·데이터가 날아가거나·신뢰가 깨지는** 핵심 사용자 플로우만 추린다. 목적은 "잘못된 수정이 기존 동작을 깨뜨리는 것(회귀)을 막는 것"이지 모든 분기 커버가 아니다.

### flaky 예방 — "PR 머지 전, 작성 단계에서 잡는다"
1. **하드 대기 금지**: `waitForTimeout(3000)` ❌ → Playwright **조건 기반 auto-wait**(`expect`의 자동 재시도) ✅. 고정 대기는 평소엔 낭비, 어느 순간 부족해지면 깨진다.
2. **시멘틱 셀렉터**: CSS 클래스명·DOM 구조 ❌ → `getByRole`/name(예: 버튼 "다음") ✅. 디자인이 바뀌어도 덜 깨진다. (→ [[spec-as-code]]로도 읽기 좋아짐)
3. **burn-in**: 작성 직후 같은 테스트를 `--repeat-each`로 여러 번 반복, 한 번이라도 실패하면 flaky로 간주. 머지 후 추적 비용 ≫ 작성 직후 잡는 비용.

### 외부 의존성 모킹 — "통제할 수 있는 것만 테스트"
- 브라우저에서 호출하는 외부 API → Playwright `page.route`로 응답 모킹.
- SSR/BFF 등 서버에서 호출하는 부분 → **E2E용 환경변수**로 고정 응답 반환하도록 분기.

### 테스트 독립성 — 상태를 API로 prefill
- 공통 앞단(약관동의·본인인증)을 매 테스트가 반복하면 느리고, 앞단이 바뀌면 전부 실패. → **테스트용 API로 "검증할 갈래의 시작점" 상태를 미리 세팅**하고 거기서부터 검증(Playwright 공식 권장: 각 테스트는 독립적).

### Trace = 에이전트의 디버깅 입력
- 테스트 실행 시 생성되는 `trace.zip`에 스크린샷 타임라인·액션 before/after·DOM 스냅샷·콘솔·네트워크가 통째로 담긴다. Trace Viewer로 사람이 보던 것을, `Playwright CLI`/`playwright-trace` 스킬로 에이전트가 탐색 → 원인 분석·수정. (사람이 DevTools로 디버깅하는 것과 동형)

### 작성·수리 자동화
- **시작은 직접 안 짜기**: `npx playwright codegen`으로 클릭하며 코드+HAR 생성 → 에이전트가 기존 컨벤션에 맞춰 정리(test.step 분리, 인증 초기화, 타입/린트 검증).
- **공식 에이전트 3종** (`npx playwright init-agents --loop=claude`): **Planner**(코드베이스 탐색→md 계획서, 사람이 검토)·**Generator**(계획서→코드, 브라우저로 실행 검증)·**Healer**(실패 테스트 trace 분석→수정→재실행 반복). 역할 분리 이유 = 한 LLM에 다 시키면 완성도↓.
- 공식 에이전트도 그냥 설치만으론 헤맨다 → 팀 하네스(AGENTS.md·e2e-testing 단일출처·유틸/헬퍼·MCP)가 전제. → [[harness-engineering]]

### CI 연동
- PR → CI에서 e2e 실행 → 실패 시 머지 차단 + PR 코멘트 + `trace.zip` 아티팩트 업로드 → 에이전트가 다운로드·디버깅(e2e-debug 스킬)·Healer 수정. → [[self-improvement-loop]]

## 관련
- [[self-improvement-loop]] — 이 센서가 닫는 회로
- [[spec-as-code]] — 테스트가 명세이기도 한 이유
- [[harness-engineering]] — 에이전트가 잘 쓰려면 필요한 환경
- [[rules/stacks/frontend]] — 컴포넌트 경계 결함 예방 FE 룰(boolean 직렬화·useEffect deps·필드 타입가드)
- [[rules/stacks/nextjs]] / [[rules/company/git-pr]] — Next.js·CI/PR 운영 룰
- 출처: [[sources/llm-agents/playwright-e2e-harness-naverpay]]
