# llm-wiki

개인 LLM 위키 (마크다운 기반, RAG 아님). Obsidian = 사람용 프론트엔드,
Claude Code = 위키를 빌드/정리하는 엔진.

## 구조
- `index.md` — 마스터 카탈로그 (모든 페이지 + summary). `/ingest` 가 자동 갱신
- `MOC/` — 주제별 목차 허브 (Maps of Content). 진입점: `MOC/Home.md`
- `notes/` — 영구 지식 (개념 1개=1파일, 내 언어로 재정리)
- `sources/` — 원천 기록 (영상/글/PDF 요약 + 출처 메타)
- `projects/` — 작업·의사결정·삽질 로그
- `rules/` — 재사용 운영 산출물(팀룰·공통 CLAUDE.md). company/ team/ stacks/
- `inbox/` — 미분류 임시
- `templates/` — frontmatter 스키마 템플릿

## frontmatter
- 공통: `type, title, summary, created, updated, visibility, tags`
- `summary`(1~2문장)로 **계층 검색**(제목/태그/summary 먼저, 본문은 필요할 때만)
- `notes/`는 `provenance: extracted|inferred|ambiguous` 로 출처/추론 구분

## 커맨드
- `/ingest <url|파일|유튜브>` — 소스를 읽어 위키화 (계층 검색 → sources + notes + 링크 + MOC + index, 중복은 병합)
- `/wiki-review` — 고아·dead-link·모순·visibility·index 정합성 점검
- `/wiki-apply <프로젝트 경로>` — rules 조합해 대상 CLAUDE.md 생성
- `/wiki-publish <대상>` — visibility:team|public 만 export (유출 검사 포함)

## 프라이버시
- 모든 노트는 frontmatter `visibility`(기본 `private`).
- 동기화(git, 개인 private 레포)와 공유(export)는 분리. 기본은 비공개.

## 설계 근거
- 패턴: Andrej Karpathy "LLM Wiki"
- 채택 기법 출처: [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian),
  [obsidian-wiki](https://github.com/ar9av/obsidian-wiki)
- 스펙: `docs/superpowers/specs/2026-06-18-llm-wiki-design.md`
