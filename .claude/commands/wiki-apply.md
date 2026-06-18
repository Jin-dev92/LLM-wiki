---
description: rules/ 조각을 조합해 대상 프로젝트의 CLAUDE.md를 생성/갱신한다
argument-hint: "<대상 프로젝트 경로>"
---

대상 경로: $ARGUMENTS

1. **대상 파악**: 대상 프로젝트의 스택/팀/회사 맥락을 파악(있으면 기존 CLAUDE.md 읽기,
   package.json/설정으로 스택 추정).
2. **조각 선택**: `rules/company/`(항상 후보), `rules/team/`, `rules/stacks/`에서
   대상에 맞는 rule 파일을 고른다. **어떤 조각을 쓸지 사용자에게 확인받는다.**
3. **조합**: 선택한 rule 본문을 출처 주석(`<!-- from rules/... -->`)과 함께 조합.
   frontmatter는 제외하고 본문 규칙만.
4. **쓰기**: 대상 경로의 `CLAUDE.md`로 복사. 기존 파일이 있으면 덮어쓰기 전 diff 보여주고 확인.
5. **보고**: 적용한 조각 목록과 대상 파일 경로 출력.

민감정보(키·비밀)가 rule에 섞여 있으면 중단하고 경고.
