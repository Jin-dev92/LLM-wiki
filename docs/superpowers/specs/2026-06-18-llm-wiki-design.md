# llm-wiki 설계 문서

- 작성일: 2026-06-18
- 상태: 승인됨 (사용자 승인 2026-06-18)
- 참조: 영상 컨셉 — Obsidian + Claude Code + Graphify "LLM Wiki" (https://www.youtube.com/watch?v=cNlvrU-KcRg)

## 배경 / 동기

대화·작업 중에 생기는 "재활용 가능한 개인 지식"이 잘 쌓이지 않는다. `CLAUDE.md` /
`AGENT.md` 만으로는 부족하다. 참조 영상은 이 문제를 **마크다운 기반 LLM 위키**로
해결한다: LLM이 일종의 "컴파일러"처럼 raw 소스(논문/영상/글)를 받아 기존 지식과의
연관성을 찾고 `[[링크]]`로 엮어 위키 문서로 정리한다. 사람은 Obsidian을
"Frontend"로 써서 입력·조회·그래프 탐색을 한다.

### LLM Wiki vs RAG (영상 비교표)

| 항목 | LLM Wiki | RAG |
|---|---|---|
| 셋업 복잡도 | 낮음 (마크다운 파일만) | 높음 (벡터DB+임베딩) |
| 인프라 | 없음 (로컬 파일) | 벡터DB+파이프라인 |
| 적합 규모 | 소~중 (컨텍스트 윈도우 내) | 대규모 |
| 검색 신뢰도 | 100% (전부 컨텍스트에) | 가변 (누락 가능) |
| 업데이트 | 마크다운 파일 편집 | 재청크+재임베딩 |
| 지식 축적 | 복리 (쌓을수록 쌓임) | 매번 재집성 |

## 범위 (이번 작업 = "1번: 내 위키")

- **목표**: 내 second-brain(범용 학습 지식) + 작업/프로젝트 지식 + 재사용 운영
  산출물(팀룰·공통 CLAUDE.md)을 한 vault에 쌓는다.
- **이후 계획(2번)**: 검증된 규칙을 재사용 가능한 도구(스킬/CLI)로 추출. 이번 범위 밖.
- 결과물은 **위키 그 자체(콘텐츠+워크플로)** 이며, 도구 소프트웨어가 아니다.

## 사용자 결정 사항 (확정)

- 담을 내용: second-brain + 작업/프로젝트 지식 (둘 다)
- 추가 용도: 회사 공통 팀룰 / 재사용 CLAUDE.md 관리 (기존 iCloud 관리 대체)
- 프론트엔드: **Obsidian** (`[[위키링크]]` + 그래프 뷰 + frontmatter)
- 입력 소스: 유튜브 영상, 웹 링크(기술글/문서), 로컬 파일(PDF/문서), 대화/메모 직접
- 동기화: **개인 Git private 레포** (나만 접근, 모든 노트가 버전이력·기기간 동기화·백업 대상)
- 프라이버시/공유: **모델 ① — 개인레포 + export 공유.** "동기화"와 "공유"를 분리.
  vault 전체를 팀과 공유하지 않는다. frontmatter `visibility`(기본 `private`)로 표시된
  것만 커맨드로 export. 안전 기본값(기본은 비공개).
- rules 배포: **커맨드로 조합·복사** (`/wiki-apply`), 팀 공유는 export (`/wiki-publish`)

## 아키텍처

```
   INGEST              COMPILE (Claude Code)            WIKI (Obsidian)
유튜브/웹/PDF/메모  →  CLAUDE.md 규칙 + 슬래시 커맨드  →  링크된 마크다운 + 그래프
                          ① 소스 읽기
                          ② 기존 노트와 연관성 검색
                          ③ source 기록 + atomic 노트 추출
                          ④ [[양방향 링크]] + MOC 갱신
                          ⑤ 사람 검토 플래그
```

- **Frontend**: Obsidian
- **Engine**: Claude Code — 이 레포의 `CLAUDE.md`가 COMPILE 규칙서
- **저장/동기화**: 이 폴더를 git private repo로

## 폴더 구조

```
llm-wiki/
├── CLAUDE.md              # COMPILE 규칙 (Claude가 따르는 위키화 지침)
├── index.md               # 마스터 카탈로그 (모든 페이지 + summary)
├── .claude/commands/      # /ingest, /wiki-apply, /wiki-publish, /wiki-review
├── templates/             # 타입별 frontmatter 템플릿
├── MOC/                   # 주제별 목차 허브 (예: AI.md, 개발.md)
├── notes/                 # 영구 지식 (개념 1개=1파일, 내 언어로 재정리)
├── sources/               # 원천 기록 (영상/글/PDF 요약 + 출처 메타)
├── projects/              # 작업·의사결정·삽질 로그
├── rules/                 # 재사용 운영 산출물 (배포용)
│   ├── company/
│   ├── team/
│   └── stacks/
├── inbox/                 # 미분류 임시
└── .obsidian/             # Obsidian 설정 (graph 등)
```

설계 철학(접근법 A): 노트의 *성격*으로 폴더를 얕게 나누고, 연결은 `[[링크]]`로 깊게.
`sources/`(원본)와 `notes/`(영구지식) 분리가 "복리로 쌓이는 지식"의 핵심. `rules/`는
배포용 운영 산출물로, 그 배경을 적은 `notes/`와 링크해 "이 룰이 왜 생겼나"를 그래프로
추적 가능.

## 노트 스키마 (frontmatter)

모든 노트 공통:

```yaml
---
type: note | source | project | rule | moc
title: ...
summary: 1~2문장 미리보기   # 계층 검색용
created: 2026-06-18
updated: 2026-06-18
visibility: private   # private(기본) | team | public
tags: [ai, llm]
---
```

- `summary`: 1~2문장 미리보기. 제목/태그/summary를 먼저 읽고 본문은 필요할 때만 여는
  **계층 검색**으로 위키가 커져도 컨텍스트 비용을 평탄하게 유지.
- `visibility`: 기본 `private`. export(`/wiki-publish`) 대상은 `team`/`public` 뿐.
  명시 안 하면 private로 간주 (안전 기본값).
- `note` 추가 필드: `provenance: extracted|inferred|ambiguous` — 출처 추출/LLM 추론/모호
  를 구분해 환각 방지·사람 검토를 데이터로 강제.
- `source` 추가 필드: `url`, `author`, `ingested_via: youtube|web|pdf|memo`
- `rule` 추가 필드: `scope: company|team|stack`, `applies_to: [react, supabase]`
- 본문은 `[[링크]]`로 다른 노트 연결
- 루트 `index.md` = 마스터 카탈로그(모든 페이지 + summary). `/ingest` 시 자동 갱신.

## 슬래시 커맨드 (워크플로)

- **`/ingest <url | 파일경로 | 유튜브링크>`**
  - 소스 종류 자동 판별 → 유튜브면 watch, 웹이면 fetch, 로컬이면 read
  - **계층 검색**: `index.md` + 제목/태그/summary 먼저 훑어 후보 좁힘
  - `sources/`에 원천 기록 생성 (출처 메타 + summary)
  - `notes/`에 atomic 영구노트 추출 (내 언어로 재구성, provenance 표시)
  - **merge-on-ingest**: 같은 개념 노트가 있으면 새로 만들지 말고 병합·출처 누적
  - 기존 노트 검색해 `[[링크]]` 연결, 관련 `MOC/` 갱신
  - `index.md` 갱신
  - 검토 필요 항목 보고
  - **대화/메모 직접 입력**: 별도 인자 없이, 대화 중 "이거 위키에 저장해" 또는 붙여넣은
    메모를 `/ingest` 가 받아 동일한 COMPILE 단계(③~⑤)를 수행 (`ingested_via: memo`).
- **`/wiki-apply <대상 프로젝트 경로>`**
  - `rules/`에서 필요한 조각(회사+팀+스택) 골라 조합
  - 대상 프로젝트 `CLAUDE.md`로 복사 (어떤 조각 쓸지 대화로 확인)
- **`/wiki-publish <대상 위치>`**
  - `visibility: team|public` 인 파일만 골라 팀 레포/공유 위치로 export.
  - **유출 방지 검사**: export 대상이 `[[링크]]`로 `private` 노트를 참조하면 경고하고
    중단 (깨진 링크·정보 유출 방지). 사람이 해소하도록 보고.
- **`/wiki-review`**
  - 고아 노트, 연결 누락, **dead-link(깨진 `[[링크]]`)**, **모순 주장(`[!contradiction]` 플래그)**,
    `index.md` 정합성, 오래된 노트 탐지 → 정리 제안 (영상의 "최신화")
  - `visibility` 누락 노트도 함께 보고 (기본 private로 동작하나 명시 권장)

## COMPILE 규칙 (CLAUDE.md 핵심 원칙)

1. **GIGO 방지**: `sources/`(원본 요약)와 `notes/`(영구지식)를 분리. 영구노트는
   베껴쓰지 않고 내 언어로 재구성.
2. **원자성**: 노트 1개 = 개념 1개. 길어지면 쪼개고 링크.
3. **연결 우선**: 새 노트는 기존 노트 최소 1개와 연결 시도 (복리 효과).
4. **중복 대신 병합**: 같은 개념은 새 파일 말고 기존 노트에 병합 (merge-on-ingest).
5. **계층 검색**: 항상 제목/태그/summary 먼저, 본문은 필요할 때만.
6. **출처 정직성**: 추출(extracted)과 추론(inferred)을 `provenance`로 구분.
7. **사람 검토**: 추측으로 단정하지 않고, 불확실하면 검토 플래그.
8. **민감정보**: `rules/`에 키·비밀·고객정보 금지. private repo라도 평문 노출 경고.
9. **안전 기본값(비공개)**: 새 노트는 `visibility: private` 가 기본. 공유는 export로만
   일어나는 명시적 행동. 동기화(git)와 공유(export)는 분리한다.

## 채택한 외부 기법 (2026-06-18 조사)

기존 오픈소스 구현 검토 후, 우리 범위(개인용·마크다운 only·RAG 없음)에 맞는 것만 채택:

- **provenance frontmatter** (extracted|inferred|ambiguous) — 환각 방지·사람 검토 강화.
  출처: [obsidian-wiki](https://github.com/ar9av/obsidian-wiki)
- **summary + 계층 검색** — 위키 확장성. 출처: [obsidian-wiki](https://github.com/ar9av/obsidian-wiki)
- **index.md 마스터 카탈로그 + 생성 전 중복 검사** —
  출처: [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian)
- **merge-on-ingest** (중복 대신 병합) — 출처: [obsidian-wiki](https://github.com/ar9av/obsidian-wiki)
- **/wiki-review 확장**: dead-link + `[!contradiction]` 플래그 —
  출처: [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian)

보류(범위 확대): `.manifest.json` 델타 추적, `/autoresearch`, 인용 기반 `/query`.
거부(스펙 상충/단일 사용자): 하이브리드 검색(BM25+임베딩), 멀티라이터 락, 방법론 모드 라우터.
패턴 출처: Andrej Karpathy "LLM Wiki".

## 검증 기준 (성공 조건)

- `/ingest`로 유튜브 1개 + 웹글 1개 투입 시:
  `sources/` 기록 + `notes/` atomic 노트 + 둘 사이 `[[링크]]` + MOC 갱신 발생.
- Obsidian 그래프 뷰에서 노드/링크가 보임.
- `/wiki-apply`가 rules 조각을 조합해 대상 `CLAUDE.md`를 생성.

## 범위 밖 (YAGNI — 나중에)

- 벡터DB/RAG/임베딩 (마크다운만 사용)
- 자체 웹앱 UI (Obsidian이 프론트엔드)
- 자동 스케줄 ingest, mp4/pptx export
- 재사용 도구(스킬/CLI)화 = 별도 2번 프로젝트

## 비고 / 가정

- 이 폴더(`~/WebstormProjects/llm-wiki`)는 아직 git 레포가 아님 → 구현 시 `git init`
  + private 원격 연결 필요.
- Claude Code의 메모리 시스템(`memory/` + `MEMORY.md` + `[[링크]]`)이 이 위키와
  동일 철학이라, 엔진 일부 패턴을 재활용 가능.
