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
- 동기화: **Git private 레포** (버전이력·되돌리기·팀 공유 PR)
- rules 배포: **커맨드로 조합·복사** (`/wiki-apply`)

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
├── .claude/commands/      # /ingest, /wiki-apply, /wiki-review
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
created: 2026-06-18
updated: 2026-06-18
tags: [ai, llm]
---
```

- `source` 추가 필드: `url`, `author`, `ingested_via: youtube|web|pdf|memo`
- `rule` 추가 필드: `scope: company|team|stack`, `applies_to: [react, supabase]`
- 본문은 `[[링크]]`로 다른 노트 연결

## 슬래시 커맨드 (워크플로)

- **`/ingest <url | 파일경로 | 유튜브링크>`**
  - 소스 종류 자동 판별 → 유튜브면 watch, 웹이면 fetch, 로컬이면 read
  - `sources/`에 원천 기록 생성 (출처 메타 포함)
  - `notes/`에 atomic 영구노트 추출 (내 언어로 재구성)
  - 기존 노트 검색해 `[[링크]]` 연결
  - 관련 `MOC/` 갱신
  - 검토 필요 항목 보고
- **`/wiki-apply <대상 프로젝트 경로>`**
  - `rules/`에서 필요한 조각(회사+팀+스택) 골라 조합
  - 대상 프로젝트 `CLAUDE.md`로 복사 (어떤 조각 쓸지 대화로 확인)
- **`/wiki-review`**
  - 고아 노트(링크 없는 것), 연결 누락, 오래된 노트 탐지 → 링크/정리 제안 (영상의 "최신화")

## COMPILE 규칙 (CLAUDE.md 핵심 원칙)

1. **GIGO 방지**: `sources/`(원본 요약)와 `notes/`(영구지식)를 분리. 영구노트는
   베껴쓰지 않고 내 언어로 재구성.
2. **원자성**: 노트 1개 = 개념 1개. 길어지면 쪼개고 링크.
3. **연결 우선**: 새 노트는 기존 노트 최소 1개와 연결 시도 (복리 효과).
4. **사람 검토**: 추측으로 단정하지 않고, 불확실하면 검토 플래그.
5. **민감정보**: `rules/`에 키·비밀·고객정보 금지. private repo라도 평문 노출 경고.

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
