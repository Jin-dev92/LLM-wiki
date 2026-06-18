# llm-wiki Foundation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Obsidian 기반 개인 LLM 위키 vault를 구축한다 — 폴더 구조 + COMPILE 규칙(CLAUDE.md) + 4개 슬래시 커맨드(`/ingest`, `/wiki-review`, `/wiki-apply`, `/wiki-publish`)로 raw 소스를 링크된 마크다운 지식으로 쌓고, 안전 기본값(비공개)으로 선택 export 한다.

**Architecture:** vault = 개인 git private 레포. 사람은 Obsidian(`[[링크]]`+그래프+frontmatter)을 프론트엔드로, Claude Code는 `CLAUDE.md`의 COMPILE 규칙과 `.claude/commands/`의 슬래시 커맨드로 INGEST→COMPILE→WIKI 파이프라인을 수행한다. 동기화(git)와 공유(export)는 분리한다.

**Tech Stack:** 마크다운 + YAML frontmatter, Obsidian, Claude Code 슬래시 커맨드(프롬프트 파일), git. 벡터DB/RAG/코드 빌드 없음.

**Spec:** `docs/superpowers/specs/2026-06-18-llm-wiki-design.md`

**검증 방식 안내:** 자동 테스트 프레임워크가 없다. 각 산출물은 (a) 파일 생성/구조를 `ls`/`grep`/read로 확인하고, (b) 커맨드는 마지막 Task 8에서 통제된 fixture와 실제 소스로 동작을 검증한다(스펙의 성공 조건).

---

## File Structure (decomposition)

생성/수정할 파일과 각 책임:

- `.gitignore` — 휘발성 Obsidian/OS 파일 제외
- `README.md` — vault 사용법·구조 개요
- `CLAUDE.md` — COMPILE 규칙서 (Claude가 따르는 위키화 지침) — **엔진의 핵심**
- 폴더 + 각 폴더 `README.md`(목적 설명 겸 .gitkeep 역할): `MOC/`, `notes/`, `sources/`, `projects/`, `rules/{company,team,stacks}/`, `inbox/`
- `templates/{note,source,project,rule,moc}.md` — frontmatter 스키마 템플릿
- `.claude/commands/ingest.md` — INGEST→COMPILE 파이프라인
- `.claude/commands/wiki-review.md` — 고아/누락/visibility 점검
- `.claude/commands/wiki-apply.md` — rules 조합→대상 프로젝트 CLAUDE.md
- `.claude/commands/wiki-publish.md` — visibility 기반 export + 유출 검사
- `MOC/Home.md` — 위키 진입점 허브

각 커맨드는 하나의 책임만 갖는 독립 프롬프트 파일. 작게 유지한다.

---

## Task 1: 레포 스캐폴딩 (폴더 + .gitignore + README)

**Files:**
- Create: `.gitignore`
- Create: `README.md`
- Create: `MOC/README.md`, `notes/README.md`, `sources/README.md`, `projects/README.md`, `inbox/README.md`
- Create: `rules/README.md`, `rules/company/README.md`, `rules/team/README.md`, `rules/stacks/README.md`

- [ ] **Step 1: .gitignore 작성**

`.gitignore`:
```gitignore
# OS
.DS_Store

# Obsidian (휘발성 작업 상태만 제외, 설정은 추적)
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.obsidian/cache
.trash/
```

- [ ] **Step 2: 루트 README.md 작성**

`README.md`:
```markdown
# llm-wiki

개인 LLM 위키 (마크다운 기반, RAG 아님). Obsidian = 사람용 프론트엔드,
Claude Code = 위키를 빌드/정리하는 엔진.

## 구조
- `MOC/` — 주제별 목차 허브 (Maps of Content). 진입점: `MOC/Home.md`
- `notes/` — 영구 지식 (개념 1개=1파일, 내 언어로 재정리)
- `sources/` — 원천 기록 (영상/글/PDF 요약 + 출처 메타)
- `projects/` — 작업·의사결정·삽질 로그
- `rules/` — 재사용 운영 산출물(팀룰·공통 CLAUDE.md). company/ team/ stacks/
- `inbox/` — 미분류 임시
- `templates/` — frontmatter 스키마 템플릿

## 커맨드
- `/ingest <url|파일|유튜브>` — 소스를 읽어 위키화 (sources + notes + 링크 + MOC)
- `/wiki-review` — 고아 노트·링크 누락·visibility 점검
- `/wiki-apply <프로젝트 경로>` — rules 조합해 대상 CLAUDE.md 생성
- `/wiki-publish <대상>` — visibility:team|public 만 export (유출 검사 포함)

## 프라이버시
- 모든 노트는 frontmatter `visibility`(기본 `private`).
- 동기화(git, 개인 private 레포)와 공유(export)는 분리. 기본은 비공개.
```

- [ ] **Step 3: 각 폴더 README.md 작성 (폴더 유지 + 목적 설명)**

`MOC/README.md`:
```markdown
# MOC — Maps of Content
주제별 목차 허브. 관련 노트들을 `[[링크]]`로 모으는 진입점. 폴더가 아닌 링크로 탐색.
```
`notes/README.md`:
```markdown
# notes — 영구 지식
개념 1개 = 1파일(atomic). 원문 복붙 금지, 내 언어로 재구성. 최소 1개 노트와 링크.
```
`sources/README.md`:
```markdown
# sources — 원천 기록
영상/글/PDF의 요약과 출처 메타(url, author, ingested_via). 여기서 notes/로 지식을 추출.
```
`projects/README.md`:
```markdown
# projects — 작업/의사결정 로그
진행 중 프로젝트, 의사결정 근거, 삽질 기록.
```
`inbox/README.md`:
```markdown
# inbox — 미분류 임시
나중에 정리할 임시 노트. /wiki-review 로 주기적으로 비운다.
```
`rules/README.md`:
```markdown
# rules — 재사용 운영 산출물 (배포용)
다른 레포에 배포되는 CLAUDE.md/팀룰. company/(회사공통) team/(팀별) stacks/(스택별).
배경·이유는 notes/에 적고 [[링크]]로 연결. 키·비밀·고객정보 금지.
```
`rules/company/README.md`:
```markdown
# rules/company — 회사 공통 룰
```
`rules/team/README.md`:
```markdown
# rules/team — 팀별 룰
```
`rules/stacks/README.md`:
```markdown
# rules/stacks — 스택별 룰 (react.md, supabase.md ...)
```

- [ ] **Step 4: 구조 검증**

Run: `find . -type d -not -path './.git*' | sort && echo '---' && ls MOC notes sources projects inbox rules rules/company rules/team rules/stacks`
Expected: 위 모든 폴더가 존재하고 각 폴더에 `README.md`가 보임.

- [ ] **Step 5: 커밋**

```bash
git add .gitignore README.md MOC/ notes/ sources/ projects/ inbox/ rules/
git commit -m "feat: vault 폴더 구조 + README + .gitignore 스캐폴딩"
```

---

## Task 2: frontmatter 스키마 템플릿

**Files:**
- Create: `templates/note.md`, `templates/source.md`, `templates/project.md`, `templates/rule.md`, `templates/moc.md`

- [ ] **Step 1: note 템플릿**

`templates/note.md`:
```markdown
---
type: note
title:
created:
updated:
visibility: private
tags: []
---

## 핵심
(개념을 내 언어로 1~3문장)

## 상세

## 관련
- [[]]
```

- [ ] **Step 2: source 템플릿**

`templates/source.md`:
```markdown
---
type: source
title:
created:
updated:
visibility: private
url:
author:
ingested_via: web   # youtube | web | pdf | memo
tags: []
---

## 요약

## 추출한 영구노트
- [[]]

## 출처 원문 메모
```

- [ ] **Step 3: project 템플릿**

`templates/project.md`:
```markdown
---
type: project
title:
created:
updated:
visibility: private
status: active   # active | done | archived
tags: []
---

## 목표

## 의사결정 로그

## 삽질/배운 것
- [[]]
```

- [ ] **Step 4: rule 템플릿**

`templates/rule.md`:
```markdown
---
type: rule
title:
created:
updated:
visibility: team   # rule은 보통 공유 대상
scope: stack       # company | team | stack
applies_to: []
tags: []
---

## 규칙

## 배경/이유
- [[]]
```

- [ ] **Step 5: moc 템플릿**

`templates/moc.md`:
```markdown
---
type: moc
title:
created:
updated:
visibility: private
tags: []
---

## 개요

## 노트
- [[]]
```

- [ ] **Step 6: 검증**

Run: `for f in note source project rule moc; do echo "== $f =="; head -3 templates/$f.md; done`
Expected: 5개 템플릿 모두 `type: <해당타입>` 으로 시작.

- [ ] **Step 7: 커밋**

```bash
git add templates/
git commit -m "feat: 노트 타입별 frontmatter 스키마 템플릿"
```

---

## Task 3: CLAUDE.md — COMPILE 규칙서

**Files:**
- Create: `CLAUDE.md`

- [ ] **Step 1: CLAUDE.md 작성 (엔진 핵심 규칙)**

`CLAUDE.md`:
```markdown
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
```

- [ ] **Step 2: 검증**

Run: `grep -E "COMPILE 단계|GIGO|안전 기본값|visibility" CLAUDE.md`
Expected: 4줄 모두 매치 (핵심 규칙 포함 확인).

- [ ] **Step 3: 커밋**

```bash
git add CLAUDE.md
git commit -m "feat: CLAUDE.md COMPILE 규칙서"
```

---

## Task 4: `/ingest` 커맨드

**Files:**
- Create: `.claude/commands/ingest.md`

- [ ] **Step 1: ingest 커맨드 작성**

`.claude/commands/ingest.md`:
```markdown
---
description: 소스(유튜브/웹/로컬파일/메모)를 읽어 위키화한다
argument-hint: "<url | 파일경로 | 유튜브링크>  (없으면 대화/메모 입력)"
---

입력: $ARGUMENTS

CLAUDE.md의 COMPILE 단계를 그대로 수행한다.

1. **소스 판별 & 읽기**
   - youtube.com / youtu.be → `watch` 스킬로 내용 추출 (ingested_via: youtube)
   - http(s) URL → WebFetch (ingested_via: web)
   - 로컬 경로(.pdf/.md/.txt/이미지) → Read (ingested_via: pdf)
   - 인자 없음/붙여넣은 텍스트 → 대화 내용을 소스로 (ingested_via: memo)
2. **연관성 검색**: `notes/`, `sources/`, `MOC/` 에서 관련 기존 노트를 grep으로 탐색.
3. **원천 기록 생성**: `templates/source.md` 기반으로 `sources/<제목-kebab>.md` 작성.
   요약 + url/author/ingested_via + created/updated(오늘) 채움.
4. **영구노트 추출**: 핵심 개념을 `templates/note.md` 기반으로 `notes/<개념-kebab>.md`로.
   **원문 복붙 금지, 내 언어로 재구성.** 개념이 여러 개면 파일도 여러 개(atomic).
5. **양방향 링크**: source ↔ note, note ↔ 기존 노트를 `[[링크]]`로 연결.
   관련 `MOC/` 파일이 있으면 항목 추가, 없고 주제가 크면 새 MOC 제안.
6. **보고**: 생성/수정한 파일 목록 + 연결한 링크 + 검토 필요 항목을 요약 출력.

기본 visibility는 private. 사용자가 공유용이라 명시하면 team/public.
```

- [ ] **Step 2: 검증 (구조)**

Run: `grep -E "COMPILE|watch|WebFetch|ingested_via|\[\[링크\]\]" .claude/commands/ingest.md | head`
Expected: 파이프라인 핵심 단계 키워드가 매치.

- [ ] **Step 3: 커밋**

```bash
git add .claude/commands/ingest.md
git commit -m "feat: /ingest 커맨드 (INGEST→COMPILE 파이프라인)"
```

---

## Task 5: `/wiki-review` 커맨드

**Files:**
- Create: `.claude/commands/wiki-review.md`

- [ ] **Step 1: wiki-review 커맨드 작성**

`.claude/commands/wiki-review.md`:
```markdown
---
description: 위키 건강검진 — 고아 노트, 링크 누락, visibility 누락, 오래된 노트 점검
---

위키를 점검하고 정리 제안을 한다. 자동 수정하지 말고 제안 후 승인받는다.

1. **고아 노트**: 어떤 파일에서도 `[[링크]]`로 참조되지 않고, 스스로도 링크가 없는
   `notes/` 파일을 찾는다. (grep으로 `[[파일명]]` 역참조 확인)
2. **링크 누락**: source 노트인데 추출된 `notes/`로의 링크가 없는 경우.
3. **visibility 누락**: frontmatter에 `visibility` 키가 없는 노트 목록.
   (없으면 private로 동작하지만 명시 권장)
4. **inbox 적체**: `inbox/`에 남은 파일 → 정리 대상.
5. **오래된 노트**: `updated` 가 오래된 핵심 노트(선택).

각 항목을 표로 보고하고, 어떤 링크/이동/정리를 할지 제안. 사용자 승인 후에만 변경.
```

- [ ] **Step 2: 검증**

Run: `grep -E "고아 노트|visibility 누락|inbox 적체|승인" .claude/commands/wiki-review.md`
Expected: 핵심 점검 항목 매치.

- [ ] **Step 3: 커밋**

```bash
git add .claude/commands/wiki-review.md
git commit -m "feat: /wiki-review 커맨드 (위키 건강검진)"
```

---

## Task 6: `/wiki-apply` 커맨드

**Files:**
- Create: `.claude/commands/wiki-apply.md`

- [ ] **Step 1: wiki-apply 커맨드 작성**

`.claude/commands/wiki-apply.md`:
```markdown
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
```

- [ ] **Step 2: 검증**

Run: `grep -E "조각 선택|확인받는다|diff|민감정보" .claude/commands/wiki-apply.md`
Expected: 조합·확인·안전 키워드 매치.

- [ ] **Step 3: 커밋**

```bash
git add .claude/commands/wiki-apply.md
git commit -m "feat: /wiki-apply 커맨드 (rules 조합→대상 CLAUDE.md)"
```

---

## Task 7: `/wiki-publish` 커맨드 (export + 유출 검사)

**Files:**
- Create: `.claude/commands/wiki-publish.md`

- [ ] **Step 1: wiki-publish 커맨드 작성**

`.claude/commands/wiki-publish.md`:
```markdown
---
description: visibility team|public 인 노트만 대상 위치로 export (유출 검사 포함)
argument-hint: "<export 대상 위치(폴더/레포 경로)>"
---

대상 위치: $ARGUMENTS

1. **공유 후보 수집**: frontmatter `visibility: team` 또는 `public` 인 파일만 수집.
   `visibility` 없거나 `private` 인 파일은 제외(안전 기본값).
2. **유출 방지 검사 (핵심)**: 수집한 각 파일의 본문 `[[링크]]` 대상을 확인.
   대상이 `private`(또는 미수집) 노트면 → **경고하고 export 중단.**
   깨진 링크/정보 유출 위험을 사람이 해소하도록 위반 목록을 보고.
3. **export**: 검사를 통과하면 수집 파일을 대상 위치로 복사(폴더 구조 유지).
4. **보고**: export한 파일 수/목록, 제외된 private 수, 발견된 위반을 요약.

동기화(git)와 공유(export)는 분리된 행위임을 명심 — 이 커맨드만 외부로 내보낸다.
```

- [ ] **Step 2: 검증**

Run: `grep -E "visibility: team|유출 방지|중단|안전 기본값" .claude/commands/wiki-publish.md`
Expected: visibility 필터 + 유출 검사 + 중단 로직 매치.

- [ ] **Step 3: 커밋**

```bash
git add .claude/commands/wiki-publish.md
git commit -m "feat: /wiki-publish 커맨드 (visibility export + 유출 검사)"
```

---

## Task 8: 진입점 MOC + 엔드투엔드 동작 검증 (acceptance)

스펙의 성공 조건을 통제된 fixture와 실제 소스로 검증한다.

**Files:**
- Create: `MOC/Home.md`
- (검증 과정에서 `sources/`, `notes/`, `MOC/` 에 실제 파일 생성)

- [ ] **Step 1: 진입점 Home MOC 작성**

`MOC/Home.md`:
```markdown
---
type: moc
title: Home
created: 2026-06-18
updated: 2026-06-18
visibility: private
tags: [home]
---

## 위키 진입점
- 주제 MOC들이 여기로 모인다.

## MOC 목록
- [[]]
```

커밋:
```bash
git add MOC/Home.md
git commit -m "feat: Home MOC 진입점"
```

- [ ] **Step 2: /ingest 웹 소스 검증**

실행: `/ingest https://www.anthropic.com/engineering` (또는 임의 기술 아티클 URL)
확인:
Run: `ls sources/ && echo '--- notes ---' && ls notes/ && echo '--- links ---' && grep -rl "\[\[" sources/ notes/`
Expected: `sources/`에 새 source 노트, `notes/`에 1개 이상 영구노트, 둘 사이 `[[링크]]` 존재.

- [ ] **Step 3: /ingest 유튜브 소스 검증**

실행: `/ingest https://www.youtube.com/watch?v=cNlvrU-KcRg`
확인:
Run: `grep -rl "ingested_via: youtube" sources/`
Expected: youtube로 들어온 source 노트가 1개 이상.

- [ ] **Step 4: /wiki-review 검증**

실행: `/wiki-review`
Expected: 고아 노트/visibility 누락/inbox 적체 등을 표로 보고하고, 변경 전 승인 요청.

- [ ] **Step 5: /wiki-apply 검증 (fixture)**

준비:
Run: `printf -- '---\ntype: rule\ntitle: react-base\nvisibility: team\nscope: stack\napplies_to: [react]\n---\n\n## 규칙\n- 함수형 컴포넌트만 사용\n' > rules/stacks/react.md && mkdir -p /tmp/wiki-apply-test`
실행: `/wiki-apply /tmp/wiki-apply-test`
확인:
Run: `cat /tmp/wiki-apply-test/CLAUDE.md`
Expected: react 규칙 본문이 출처 주석과 함께 포함된 CLAUDE.md 생성. (frontmatter 미포함)
정리: `rm -rf /tmp/wiki-apply-test`

- [ ] **Step 6: /wiki-publish 유출 검사 검증 (fixture)**

준비 (team 노트가 private 노트를 링크하는 위반 상황):
Run: `printf -- '---\ntype: note\ntitle: 비밀메모\nvisibility: private\n---\n\n비공개\n' > notes/secret-memo.md && printf -- '---\ntype: note\ntitle: 공유노트\nvisibility: team\n---\n\n참조 [[secret-memo]]\n' > notes/shared-note.md && mkdir -p /tmp/wiki-pub-test`
실행: `/wiki-publish /tmp/wiki-pub-test`
Expected: shared-note가 private인 secret-memo를 링크하므로 **경고하고 중단**, 위반 목록 보고. `/tmp/wiki-pub-test`는 비어 있음.
정리: `rm -f notes/secret-memo.md notes/shared-note.md && rm -rf /tmp/wiki-pub-test`

- [ ] **Step 7: Obsidian 그래프 수동 확인 (사람)**

vault를 Obsidian으로 열어 그래프 뷰에 노드/링크가 보이는지 확인. (수동, 자동화 불가)

- [ ] **Step 8: 검증 산출물 커밋**

```bash
git add -A
git commit -m "test: /ingest 실제 소스로 엔드투엔드 동작 검증"
```

---

## 비고
- 이 폴더는 이미 `git init` 완료. 원격은 **private repo** 로 연결할 것(민감정보 보호).
- Obsidian은 vault를 열면 `.obsidian/` 설정을 자동 생성한다(workspace는 .gitignore됨).
- 2번(재사용 도구화)은 별도 스펙/플랜. 이 플랜 범위 밖.
