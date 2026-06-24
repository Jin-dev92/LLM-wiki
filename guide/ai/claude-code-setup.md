---
type: project
title: Claude Code 환경 셋업 (스킬·플러그인 온보딩)
summary: 다른 환경에서 현재 Claude Code 환경을 재현하기 위한 온보딩 체크리스트 — 플러그인 4개, gstack 스킬 스위트, MCP 커넥터, 추천 확장 도구, 트러블슈팅, 설치 명령.
created: 2026-06-18
updated: 2026-06-24
visibility: private
status: active
tags: [meta, claude-code, onboarding, setup, sync]
---

## 목표
새 기기/계정에서 지금과 같은 Claude Code 작업 환경을 빠르게 복원한다.
아래 목록대로 설치하면 동일한 스킬/플러그인을 쓸 수 있다.

> 출처: `~/.claude`의 실제 설정에서 추출 + iCloud `claude/guides/ai` 가이드로 교차검증.
> 가이드 원문은 [[claude-code-core-tools-guide]], [[claude-code-gstack-superpowers-guide]] 참고.

---

## 1. 플러그인 (마켓플레이스 경유)
현재 user 스코프로 설치된 플러그인 4개. (출처: `~/.claude/plugins/installed_plugins.json`, `known_marketplaces.json`)

| 플러그인 | 마켓플레이스 | GitHub repo | 버전 |
|---|---|---|---|
| `codex` | openai-codex | `openai/codex-plugin-cc` | 1.0.4 |
| `superpowers` | superpowers-marketplace | `obra/superpowers-marketplace` | 5.1.0 |
| `claude-md-management` | claude-plugins-official | `anthropics/claude-plugins-official` | 1.0.0 |
| `watch` | claude-video | `bradautomates/claude-video` | 0.1.3 |

**설치** (Claude Code 안에서 슬래시 명령):
```text
/plugin marketplace add openai/codex-plugin-cc
/plugin marketplace add obra/superpowers-marketplace
/plugin marketplace add anthropics/claude-plugins-official
/plugin marketplace add bradautomates/claude-video

/plugin install codex@openai-codex
/plugin install superpowers@superpowers-marketplace
/plugin install claude-md-management@claude-plugins-official
/plugin install watch@claude-video
```

- `superpowers`는 공식 마켓플레이스로도 설치 가능: `/plugin install superpowers@claude-plugins-official` (현재 설치본은 community = `obra/...`).
- `watch`는 영상 자막/Whisper 처리에 `GROQ_API_KEY`(권장) 또는 `OPENAI_API_KEY`가
  `~/.config/watch/.env`에 필요하다.
- `codex`는 로컬 Codex CLI가 있어야 일부 기능이 동작한다(`/codex:setup`으로 확인).

---

## 2. gstack 스킬 스위트
`~/.claude/skills/`에 깔린 ~55개 standalone 스킬(browse·qa·ship·review·design-*·
ios-*·investigate·spec·share-tech 등)은 전부 **gstack** 한 레포에서 온다.

- repo: `https://github.com/garrytan/gstack.git`
- 설치 위치: `~/.claude/skills/gstack`
- 현재 버전: `1.48.0.0` (`~/.claude/skills/gstack/VERSION`)

**설치** (iCloud 가이드로 검증됨):
```sh
# 얕은 클론 권장
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup
```
- 설치 확인: `/office-hours`(gstack) · `/brainstorming`(superpowers) 인식되면 성공.
- 설치 후 업데이트는 `/gstack-upgrade` 스킬로.
- 브라우저 기반 스킬(browse·qa 등)은 별도 Chromium/확장 셋업이 필요할 수 있다(`/connect-chrome`).

---

## 3. MCP 커넥터 (계정 연동)
claude.ai 계정에 연동된 MCP 서버들(도구로 노출됨). 새 환경에선 각 서비스 재인증 필요.
- Notion / Gmail / Google Calendar / Google Drive
- 인증: 해당 도구 첫 호출 시 `authenticate` → `complete_authentication` 흐름.
- 코드용 MCP 설치 가이드는 함께 보관함 → `guide/mcp/serena-mcp-setup-guide.md`(LSP 코드 인텔리전스),
  `guide/mcp/aws-mfa-mysql-mcp-guide.md`(RDS 접근 + MFA).

---

## 4. 레포-로컬 스킬 (이 위키 전용)
`llm-wiki` 레포의 `.claude/`에 포함되어 레포만 클론하면 따라온다(별도 설치 불필요):
- `ingest` · `wiki-apply` · `wiki-publish` · `wiki-review`

---

## 5. 추천 확장 도구 (현재 미설치)
실무 추천이지만 아직 안 깐 것들. 필요 시 설치. (출처: [[claude-code-core-tools-guide]])

| 도구 | 언제/왜 | 설치 |
|---|---|---|
| **graphify** ⭐ | **입사 직후 낯선 코드베이스 빠른 파악** — 폴더(코드·문서·논문)를 지식그래프로(커뮤니티 탐지·god node·경로/explain 질의). 위키 provenance(EXTRACTED/INFERRED/AMBIGUOUS)·GIGO 철학과 정렬 | `uv tool install graphifyy` → `graphify install` (레포 한정: `graphify install --project`) |
| Matt Pocock Skills | PR 전 코드 리뷰·아키텍처 개선 (`tdd`·`grill-me`·`diagnose` 등) | `npx skills@latest add mattpocock/skills` → `/setup-matt-pocock-skills` |
| Vercel React Best Practices | React/Next.js 성능·서버컴포넌트 패턴 강제 | `npx skills add vercel-labs/agent-skills` |
| PPTX / PDF Skills | 보고서·슬라이드 자동 생성 | `npx skills add skills-ai/presentation-tools` |

> **graphify 주의점** (도입 전 확인): ⓐ Python + `uv`(권장) 필요 — Mac에선 plain `pip` 피할 것. ⓑ 의미 추출(semantic)은 LLM 토큰 비용 발생, 큰 레포는 서브에이전트 병렬 처리. ⓒ gstack의 `gbrain`(코드 지식 레이어)과 **기능이 일부 중복** → 입사 환경에서 둘 중 무엇을 쓸지 한 번 비교. ⓓ repo: `safishamsi/graphify`, PyPI 패키지명은 `graphifyy`(y 두 개), CLI는 `graphify`. 스킬은 `~/.claude/skills/graphify/`에 등록됨. 분석 근거: 4개 skills 레포 분석(2026-06-20 대화) (분석만, 미설치 상태).

---

## 5-1. cherry-pick 설치한 개별 스킬 (agent-skills 발췌, 2026-06-20)
`addyosmani/agent-skills`로 **전환하지 않고**(→ `docs/decisions/ADR-0001`), 고유값 스킬만 발췌해
글로벌 `~/.claude/skills/`에 복사. gstack 디렉터리와 형제라 `/gstack-upgrade`가 건드리지 않음.
slash 명령 아닌 auto-activate 스킬이라 gstack과 충돌 없음.

| 스킬 | 왜 | 비고 |
|---|---|---|
| `documentation-and-adrs` | 로컬에 ADR *작성* 기능 부재 → 공백 보충 | gstack `document-generate`는 ADR 읽기만 |
| `doubt-driven-development` | 작업 중 적대적 fresh-context 리뷰(보안·민감 로직) | `agents/` 로스터 없이 설치 → generic 서브에이전트로 degrade(자체 fallback 보유) |
| `source-driven-development` (+ `hooks/sdd-cache-*`) | 공식문서 grounding으로 할루시네이션 방지 | ✅ 훅 배선 완료(2026-06-20): `~/.claude/settings.json`의 PreToolUse/PostToolUse(matcher `WebFetch`)에 등록. 복원 시 동일 설정 필요 |

**복원**(새 환경): 위 경로에 `addyosmani/agent-skills`의 동일 파일을 복사하면 됨(upstream 자동 동기화 없음 — 수동 갱신).

---

## 5-2. 감사 로깅 훅 (cc-audit-hooks, 2026-06-24)
Claude Code 프롬프트·도구 호출을 JSONL로 로깅하고 위험 Bash 명령을 PreToolUse에서 차단하는
자작 훅. **소스 = private repo `Jin-dev92/cc-audit-hooks`**, 라이브 = `~/.claude/hooks/audit/`.
표준 라이브러리만, 훅은 세션을 깨지 않음(예외 삼키고 exit 0).

**복원**:
```sh
git clone https://github.com/Jin-dev92/cc-audit-hooks.git ~/WebstormProjects/cc-audit-hooks
cd ~/WebstormProjects/cc-audit-hooks && python3 install.py
```
- `install.py`가 `~/.claude/hooks/audit/` 배치 + `settings.json`의 UserPromptSubmit/PreToolUse/PostToolUse에 **기존 보존하며** 병합 등록(§5-1의 SDD WebFetch 훅과 공존). 멱등.
- 로그: `~/.claude/logs/audit/*.jsonl`(0600) + `index.db`. 리포트: `python3 ~/.claude/hooks/audit/audit_report.py`.
- 차단 규칙: `~/.claude/hooks/audit/danger_rules.json`. ⚠️ 레닥션은 휴리스틱 — 로그 파일 공유·커밋 금지.
- 상세 복원 가이드: [[audit-logging-setup]]. 동기: [[ai-experience-on-resume]] tip 3.

---

## 6. 함께 보관한 참고 가이드 (`guide/` 하위)
iCloud `claude/guides`에서 가져온 인프라·MCP·워크플로우 설정 가이드(원문 사본).
- `guide/mcp/serena-mcp-setup-guide.md` — Serena LSP MCP 설치
- `guide/mcp/aws-mfa-mysql-mcp-guide.md` — AWS MFA + MySQL MCP 설정
- `guide/aws/rds-snapshot-restore-guide.md` — RDS 스냅샷 → 로컬 MySQL 복구
- `guide/github-actions/pr-workflow-setup-guide.md` — Claude PR 자동 리뷰 워크플로우 구축
- `guide/github-actions/pr-workflow-summary.md` — 위 워크플로우 요약(v1.1)
> 제외: `pr-workflow-reference`(602줄)·`pr-workflow-execution-analysis`(862줄) — 특정 프로젝트 전용/중복이라 미반입. 필요하면 추가.

---

## 트러블슈팅 (gstack + superpowers)
출처: [[claude-code-gstack-superpowers-guide]]

| 증상 | 원인 | 해결 |
|---|---|---|
| `/office-hours` 미인식 | CLAUDE.md에 gstack 섹션 누락 | Claude Code 세션 재시작 |
| Superpowers 스킬 미작동 | 훅 스크립트 권한 | `chmod +x ~/.claude/plugins/superpowers/hooks/session-start.sh` |
| Worktree 오류 | git 버전 / stale worktree | `git worktree prune` 후 재시도 |
| 캐시로 재설치 필요 | - | `/plugin install superpowers@superpowers-marketplace --force` |

---

## 동기화 규칙
- **원본(source of truth)**: 로컬 `~/.claude` 실제 설정 + iCloud `claude/guides`.
- **여기**: git으로 버전관리하는 온보딩 스냅샷. 방향은 **환경/iCloud → 위키** 단방향.
- 플러그인/스킬을 추가·제거·업데이트하면 이 문서를 갱신한다. 현재 상태 덤프:

  ```sh
  cat ~/.claude/plugins/installed_plugins.json
  cat ~/.claude/plugins/known_marketplaces.json
  cat ~/.claude/skills/gstack/VERSION
  ls ~/.claude/skills
  ```
- `guide/{mcp,aws,github-actions}/`의 가이드는 iCloud 원문 사본이라, iCloud가 바뀌면 다시 복사해 갱신한다.
- 위 출력과 이 문서의 표를 비교해 차이가 있으면 표/버전을 수정하면 된다.
  (Claude에게 "claude code 셋업 문서 동기화해줘"라고 해도 됨.)

## 의사결정 로그
- 2026-06-18: gstack은 플러그인이 아니라 단일 git repo(`garrytan/gstack`)에서 온
  standalone 스킬 묶음임을 확인 → 플러그인 표와 분리해 별도 섹션으로 기록.
- 2026-06-18: gstack 설치 명령을 iCloud `guides/ai` 가이드로 교차검증해 "검증됨"으로 승격(이전엔 inferred).
- 2026-06-18: iCloud `guides/ai` 2개를 sources/에 원문 보관, `guides/{mcp,aws,github-actions}` 5개를 `guide/` 하위로 반입.
- 2026-06-18: 글로벌 지침 사본을 `guide/CLAUDE.md` → `guide/claude-global.md`로 개명(하위 CLAUDE.md 자동로드 방지).
- 2026-06-18: 주제별 정리를 위해 `claude-code-setup.md`·`claude-global.md`를 `guide/ai/`로 이동(iCloud `guides/ai` 구조와 일치).
- 2026-06-20: 추천 확장 도구에 **graphify**(`safishamsi/graphify`) 추가 — 입사 직후 낯선 코드베이스 온보딩용으로 판단. **문서에만 기재, 실제 설치 안 함**(사용자 지시). gbrain과 기능 중복은 도입 시 비교하기로. 4개 skills 레포 분석 결과 graphify가 이 위키 철학과 가장 정렬됨(4개 skills 레포 분석(2026-06-20 대화)).
- 2026-06-20: `addyosmani/agent-skills` 전체 분석 후 **전환하지 않기로 결정**(→ `docs/decisions/ADR-0001`). gstack+superpowers가 실행 도구 측면에서 상위집합이라 전환은 순손실. 대신 `documentation-and-adrs`·`doubt-driven-development`·`source-driven-development` 3종을 cherry-pick해 글로벌 설치(§5-1).
- 2026-06-24: 자작 **감사 로깅 훅 cc-audit-hooks** 구축·설치(§5-2). 생각등대 영상 tip 3([[ai-experience-on-resume]]) 실천 — 프롬프트·도구 로깅 + 위험명령 차단. private repo `Jin-dev92/cc-audit-hooks`, brainstorming→writing-plans→subagent-driven으로 TDD 구현(29 tests), opus 전체 리뷰 후 rm 분리플래그 우회·installer 크래시 수정. 라이브 설치 완료(settings.json 백업).

## 삽질/배운 것
- `~/.claude/skills`의 개별 스킬을 하나하나 설치한 게 아니라 gstack 레포 하나가
  전부를 제공한다. 온보딩 시 레포 clone + setup 한 번이면 스킬 대부분이 복원된다.
