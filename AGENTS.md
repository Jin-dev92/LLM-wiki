# llm-wiki — COMPILE 규칙

이 레포는 마크다운 기반 개인 LLM 위키다. 너(Codex)는 raw 소스를 받아
기존 지식과 엮어 링크된 위키로 "컴파일"하는 엔진이다. Obsidian이 사람용 프론트엔드다.

## 폴더 역할
- `MOC/` 주제 목차 허브 / `notes/` 영구 지식(atomic) / `sources/` 원천 기록
- `projects/` 작업 로그 / `rules/` 배포용 운영 산출물 / `inbox/` 미분류
- `guide/` 환경 재현·온보딩 참조 문서(글로벌 설정 사본, Codex 셋업 등). 작업 로그가 아니라 "복원용 가이드".
- `index.md` 루트의 마스터 카탈로그(모든 페이지 목록 + summary). 노트 생성/병합 시 갱신.
- 새 파일은 `templates/`의 해당 타입 frontmatter를 따른다.

## frontmatter 필수
모든 노트는 `type, title, summary, created, updated, visibility, tags`. 날짜는 `YYYY-MM-DD`.
- `summary`: 1~2문장 미리보기(계층 검색용).
- `visibility` 미기재 시 `private`로 간주.
- `notes/`는 `provenance: extracted|inferred|ambiguous` 추가 — 출처 추출/LLM 추론/모호 구분.

## COMPILE 단계 (소스 → 위키)
1. **소스 읽기**: 유튜브=watch 스킬, 웹=WebFetch, 로컬=Read, 메모=대화 내용.
2. **연관성 검색(계층)**: 먼저 `index.md`와 각 노트의 제목/태그/summary를 훑어 후보를
   좁힌 뒤, 필요한 노트만 본문을 연다. (위키가 커져도 컨텍스트 비용 평탄)
3. **원천 기록**: `sources/`에 source 노트 생성(summary + url/author/ingested_via).
4. **영구노트 추출/병합**: 핵심 개념을 `notes/`에 atomic하게 — **원문 복붙 금지, 내 언어로 재구성**.
   **이미 같은 개념 노트가 있으면 새로 만들지 말고 기존에 병합**(merge-on-ingest),
   출처는 frontmatter/본문에 누적. 추론한 문장은 `provenance: inferred`로 표시.
5. **양방향 링크**: 새 노트 ↔ 기존 노트를 `[[링크]]`로 연결. 관련 `MOC/` 갱신.
6. **카탈로그 갱신**: 생성/병합한 페이지를 `index.md`에 반영.
7. **검토 플래그**: 불확실하면 단정하지 말고 검토 필요 항목을 보고.

## 핵심 원칙
- **GIGO 방지**: `sources/`(원본 요약)와 `notes/`(영구지식)를 분리. notes는 재구성.
- **원자성**: 노트 1개 = 개념 1개. 길어지면 쪼개고 링크.
- **연결 우선**: 새 노트는 기존 노트 최소 1개와 연결 시도(복리).
- **중복 대신 병합**: 같은 개념은 새 파일 말고 기존 노트에 병합.
- **계층 검색**: 항상 제목/태그/summary 먼저, 본문은 필요할 때만.
- **출처 정직성**: 추출(extracted)과 추론(inferred)을 `provenance`로 구분. 답변은 위키 근거.
- **사람 검토**: 추측으로 단정 금지. 모르면 묻는다.
- **민감정보**: `rules/`에 키·비밀·고객정보 금지(평문 노출 경고).
- **안전 기본값(비공개)**: 새 노트 기본 `visibility: private`. 공유는 export로만.
  동기화(git)와 공유(export)는 분리한다.
