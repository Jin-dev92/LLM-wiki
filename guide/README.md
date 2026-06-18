# guide — Claude 공통 환경 동기화 / 온보딩

여러 프로젝트에 걸쳐 공통으로 쓰는 Claude 설정과 환경 재현 가이드를 git으로
버전관리/백업하는 폴더. 다른 환경에서 온보딩할 때 여기를 보면 환경을 복원할 수 있다.

## 파일·폴더
- `ai/claude-global.md` — **글로벌 `~/.claude/CLAUDE.md`의 사본** (모든 프로젝트 공통 지침).
  - 파일명을 `CLAUDE.md`로 두면 하위 디렉토리 지침으로 자동 로드되므로 `claude-global.md`로 개명함.
- `ai/claude-code-setup.md` — 플러그인·gstack 스킬·MCP·추천도구 온보딩 → [[claude-code-setup]]
- `mcp/` — Serena LSP MCP, AWS MFA+MySQL MCP 설정 가이드 (iCloud 원문 사본)
- `aws/` — RDS 스냅샷 → 로컬 MySQL 복구 가이드 (iCloud 원문 사본)
- `github-actions/` — Claude PR 자동 리뷰 워크플로우 설정·요약 (iCloud 원문 사본)

---

## 동기화 규칙
- **원본(source of truth)**:
  - `ai/claude-global.md` ← `~/.claude/CLAUDE.md`
  - `mcp/`·`aws/`·`github-actions/` ← iCloud `claude/guides/*`
- **여기**: git으로 버전관리/백업하는 사본. 방향은 **글로벌·iCloud → 위키** 단방향.
  여기서 편집하지 말고 원본을 고친 뒤 다시 복사한다.

  ```sh
  ICLOUD="$HOME/Library/Mobile Documents/com~apple~CloudDocs/claude"
  cp ~/.claude/CLAUDE.md guide/ai/claude-global.md
  cp "$ICLOUD/guides/mcp/"*.md           guide/mcp/
  cp "$ICLOUD/guides/aws/"*.md           guide/aws/
  cp "$ICLOUD/guides/github-actions/pr-workflow-setup-guide.md" guide/github-actions/
  cp "$ICLOUD/guides/github-actions/pr-workflow-summary.md"     guide/github-actions/
  ```

## 메모
- 마지막 동기화: 2026-06-18
- 이 폴더의 `*.md` 사본들은 위키 노트(frontmatter 규격)가 아니라 **원문 보관**이다. ingest 대상 아님.
- iCloud `guides/ai`의 Claude Code 도구 가이드 2개는 sources/에 원문 보관 +
  [[claude-code-setup]]으로 재구성했다.
