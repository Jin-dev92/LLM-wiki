---
description: 대상 프로젝트의 CLAUDE.md에 쌓인 컨벤션을 위키 rules/ 조각으로 수확(추출·병합)한다. wiki-apply의 역방향.
argument-hint: "[대상 프로젝트 경로 (생략 시 현재 디렉토리)]"
---

대상 경로: $ARGUMENTS  (비어 있으면 현재 작업 디렉토리)

위키 경로는 글로벌 CLAUDE.md에 고정된 `~/WebstormProjects/llm-wiki`를 사용한다.
이 명령은 **대상 프로젝트 안에서 실행**되어 그 프로젝트의 `CLAUDE.md`를 읽고, 위키의
`rules/`로 써넣는다. (wiki-apply는 위키 안에서 실행 — 정확히 반대 방향)

1. **읽기**: 대상 경로의 `CLAUDE.md`를 읽고, `@AGENTS.md` 같은 import가 있으면 따라가
   본문까지 읽는다. `package.json`/설정으로 스택을 추정(분류용).
2. **순-신규 필터**: `<!-- from rules/... -->`로 마킹된 섹션은 위키에서 나간 것이므로
   **건너뛴다**(왕복 중복 방지). 마킹 없는 "이 프로젝트 고유" 규칙만 수확 후보로 남긴다.
3. **분류**: 각 후보를 `company`(조직 공통) / `team` / `stacks/<스택>` 중 어디에 둘지
   추정하고 **사용자에게 확인받는다.**
4. **중복 대비 병합**: 후보를 위키 `index.md`와 기존 `rules/`의 제목/태그/summary와 대조한다
   (계층 검색). **이미 같은 개념의 rule이 있으면 새로 만들지 말고 그 파일에 병합** 제안,
   없으면 신규 조각. **쓰기 전 반드시 diff를 보여주고 확인받는다.**
5. **쓰기**: `templates/rule.md`의 frontmatter를 따른다(type, title, summary, created,
   updated, visibility, scope, applies_to, tags). 본문은 **원문 복붙이 아닌 재구성**.
   출처에 대상 프로젝트 경로를 누적(`source_file`/본문 주석). 신규 조각은 `~/WebstormProjects/llm-wiki/index.md`
   카탈로그에 추가한다. 날짜는 오늘(`YYYY-MM-DD`).
6. **보고**: 수확(신규)/병합/스킵한 항목 목록 + 변경한 파일 경로를 출력한다.

## 안전장치
- **민감정보 차단**: 키·비밀·고객정보가 섞여 있으면 **중단하고 경고**한다. `rules/`에 평문
  비밀을 절대 넣지 않는다.
- **visibility 기본값**: 신규 rule 조각은 기존 `rules/`와 동일하게 `team`을 기본으로 제안하고
  확인받는다(공유용 building block). 더 민감하면 `private`.
- **사람 검토**: 추측으로 단정하지 말고 모호하면 묻는다. 파일을 덮어쓰기/삭제하기 전 확인한다.
