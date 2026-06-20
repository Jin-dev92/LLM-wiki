# ADR-0001: Claude Code 워크플로우 기반을 gstack + superpowers로 유지 (agent-skills로 전환하지 않음)

## Status
Accepted

## Date
2026-06-20

## Context
현재 Claude Code 환경의 "개발 방법론 + 실행 도구" 기반은 두 축이다:
- **superpowers**(플러그인): 방법론 척추 — brainstorming, TDD, debugging, writing/executing-plans, verification 등 *어떻게(HOW)*.
- **gstack**(`~/.claude/skills/gstack`, ~56 스킬): 실행 도구체인 — 헤드리스 브라우저 QA(browse/qa), iOS 실기기 QA, design-review/shotgun, ship/land-and-deploy/canary, gbrain, plan-review 위원회 등 *무엇/실행*.

대안으로 `addyosmani/agent-skills`(단일 플러그인, 24 skills + 4 agents + hooks, 멀티플랫폼)를 전체 분석하고 기반을 이쪽으로 **전환**할지 검토했다. 셋을 동시에 쓰는 것은 `/spec /review /ship /test /build` 슬래시명령 충돌과 방법론 척추 중복(using-agent-skills ↔ using-superpowers)으로 불가능하므로 "현행 vs agent-skills" 택1 구도다.

판단 기준: 본인 실무는 풀스택(NestJS/Prisma/React, THUB CRM/AI) + 모바일 + 이력서 + 이 위키. 즉 **실제로 돌아가는 실행 도구**의 가치가 크다.

## Decision
**gstack + superpowers를 기반으로 유지한다.** agent-skills로 전환하지 않는다. 대신 agent-skills의 고유 가치 스킬만 **cherry-pick**해 글로벌(`~/.claude/skills/`)에 개별 설치한다:
- `documentation-and-adrs` (로컬에 ADR *작성* 기능 부재 → 공백 보충, 이 ADR이 첫 사용 사례)
- `doubt-driven-development` (작업 중 적대적 fresh-context 리뷰 — THUB AI/보안 민감 로직용)
- `source-driven-development` + `sdd-cache` 훅 (문서 grounding으로 할루시네이션 방지 + WebFetch 304 재검증 캐시)

## Alternatives Considered

### A. agent-skills로 전면 전환
- Pros: 단일 저자의 일관된 SDLC, 엄격한 skill anatomy, 멀티플랫폼(Gemini/Cursor/Copilot/OpenCode) 이식성, 영리한 훅(sdd-cache·simplify-ignore).
- Cons: **실행 도구 0개**(순수 산문 + 수동 훅). gstack의 브라우저/기기/배포/gbrain을 전부 상실. 방법론은 superpowers와 중복. 명령어 충돌. 전환·재학습 비용.
- Rejected: 본인 실무 기준 **부분집합**이라 순손실. 방법론 척추를 동급으로 갈아끼우며 실행 도구 전체를 버리는 거래.

### B. 셋 다 설치
- Rejected: 슬래시명령 충돌 + 두 방법론 척추 상충 + context 팽창.

### C. 현행 유지 + cherry-pick (채택)
- Pros: 검증된 환경 보존(전환비용 0), agent-skills의 진짜 고유값(ADR·doubt·source)만 흡수, 충돌 없음(auto-activate 스킬).
- Cons: cherry-pick 스킬은 수동 동기화 필요(upstream 업데이트 자동 반영 안 됨). sdd-cache 훅은 `settings.json` 수동 배선 필요.

## Consequences
- `~/.claude/skills/`에 standalone 스킬 3개 추가됨(gstack 디렉터리와 형제, `/gstack-upgrade`가 건드리지 않음). 온보딩 문서 [[guide/ai/claude-code-setup]] 갱신 대상.
- sdd-cache 훅은 복사만 됨 → 동작시키려면 `settings.json`의 `PreToolUse`/`PostToolUse`(matcher: `WebFetch`)에 수동 등록 필요(미등록 시 source-driven 스킬은 정상 동작, 캐시만 비활성).
- doubt-driven은 `agents/` persona 로스터 없이 설치 → cross-examination은 generic 서브에이전트로 degrade(스킬이 자체 fallback 보유).
- **전환 재검토 조건(flip condition)**: 주 작업 에이전트가 Claude Code가 아니게 될 때(Gemini CLI/Cursor/Codex/OpenCode 1순위). 그 환경에선 gstack(Claude 전용)이 안 돌아 agent-skills의 멀티플랫폼 이식성이 결정적이 된다 → 그때 본 ADR을 superseding ADR로 갱신.

## 근거 자료
- 분석 출처: [[sources/claude-code/agent-skills-repos-analysis]] (4개 레포 분석 + 본 결정 메모)
- 관련 노트: [[agent-skill-archetypes]] (워크플로우 묶음 아키타입끼리 택1 규칙), [[agent-skill-authoring]]
- 환경 현황: [[guide/ai/claude-code-setup]]
