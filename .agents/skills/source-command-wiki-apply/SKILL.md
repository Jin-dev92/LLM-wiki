---
name: "source-command-wiki-apply"
description: "rules/ 조각을 조합해 대상 프로젝트의 AGENTS.md 또는 CLAUDE.md를 생성/갱신한다"
---

# source-command-wiki-apply

Use this skill when the user asks to run the migrated source command `wiki-apply`.

## Command Template

대상 경로: 사용자가 제공한 대상 프로젝트 경로.

1. **대상 파악**: 대상 프로젝트의 스택/팀/회사 맥락을 파악한다. 있으면 기존 `AGENTS.md`,
   `CLAUDE.md`, `package.json`, 설정 파일을 읽어 스택을 추정한다.
2. **조각 선택**: `rules/company/`(항상 후보), `rules/team/`, `rules/stacks/`에서
   대상에 맞는 rule 파일을 고른다. **어떤 조각을 쓸지 사용자에게 확인받는다.**
3. **조합**: 선택한 rule 본문을 출처 주석(`<!-- from rules/... -->`)과 함께 조합한다.
   frontmatter는 제외하고 본문 규칙만 사용한다.
4. **쓰기**: Codex 대상이면 `AGENTS.md`, Claude Code 대상이면 `CLAUDE.md`로 적용한다.
   기존 파일이 있으면 덮어쓰기 전 diff를 보여주고 확인한다.
5. **보고**: 적용한 조각 목록과 대상 파일 경로를 출력한다.

민감정보(키, 비밀, 고객정보)가 rule에 섞여 있으면 중단하고 경고한다.
