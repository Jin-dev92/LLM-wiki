# llm-wiki — COMPILE 규칙

이 레포는 마크다운 기반 개인 LLM 위키다. 너(Claude)는 raw 소스를 받아
기존 지식과 엮어 링크된 위키로 "컴파일"하는 엔진이다. Obsidian이 사람용 프론트엔드다.

## 폴더 역할
- `MOC/` 주제 목차 허브 / `notes/` 영구 지식(atomic) / `sources/` 원천 기록
- `projects/` 작업 로그 / `rules/` 배포용 운영 산출물 / `inbox/` 미분류
- 새 파일은 `templates/`의 해당 타입 frontmatter를 따른다.

## frontmatter 필수
모든 노트는 `type, title, created, updated, visibility, tags`. 날짜는 `YYYY-MM-DD`.
`visibility` 미기재 시 `private`로 간주.

## COMPILE 단계 (소스 → 위키)
1. **소스 읽기**: 유튜브=watch 스킬, 웹=WebFetch, 로컬=Read, 메모=대화 내용.
2. **연관성 검색**: 위키 전체에서 관련 기존 노트를 검색(grep/파일 탐색).
3. **원천 기록**: `sources/`에 source 노트 생성(요약 + url/author/ingested_via).
4. **영구노트 추출**: 핵심 개념을 `notes/`에 atomic하게 — **원문 복붙 금지, 내 언어로 재구성**.
5. **양방향 링크**: 새 노트 ↔ 기존 노트를 `[[링크]]`로 연결. 관련 `MOC/` 갱신.
6. **검토 플래그**: 불확실하면 단정하지 말고 검토 필요 항목을 보고.

## 핵심 원칙
- **GIGO 방지**: `sources/`(원본 요약)와 `notes/`(영구지식)를 분리. notes는 재구성.
- **원자성**: 노트 1개 = 개념 1개. 길어지면 쪼개고 링크.
- **연결 우선**: 새 노트는 기존 노트 최소 1개와 연결 시도(복리).
- **사람 검토**: 추측으로 단정 금지. 모르면 묻는다.
- **민감정보**: `rules/`에 키·비밀·고객정보 금지(평문 노출 경고).
- **안전 기본값(비공개)**: 새 노트 기본 `visibility: private`. 공유는 export로만.
  동기화(git)와 공유(export)는 분리한다.
