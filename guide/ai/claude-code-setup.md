---
type: project
title: Claude Code 환경 셋업 (스킬·플러그인 온보딩)
summary: 다른 환경에서 현재 Claude Code 환경을 재현하기 위한 온보딩 체크리스트 — 플러그인 4개, gstack 스킬 스위트, MCP 커넥터, 추천 확장 도구, 트러블슈팅, 설치 명령.
created: 2026-06-18
updated: 2026-06-18
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
| Matt Pocock Skills | PR 전 코드 리뷰·아키텍처 개선 (`tdd`·`grill-me`·`diagnose` 등) | `npx skills@latest add mattpocock/skills` → `/setup-matt-pocock-skills` |
| Vercel React Best Practices | React/Next.js 성능·서버컴포넌트 패턴 강제 | `npx skills add vercel-labs/agent-skills` |
| PPTX / PDF Skills | 보고서·슬라이드 자동 생성 | `npx skills add skills-ai/presentation-tools` |

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

## 삽질/배운 것
- `~/.claude/skills`의 개별 스킬을 하나하나 설치한 게 아니라 gstack 레포 하나가
  전부를 제공한다. 온보딩 시 레포 clone + setup 한 번이면 스킬 대부분이 복원된다.
