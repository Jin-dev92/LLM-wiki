---
type: rule
title: Git/PR + 공통 프로세스 규칙 (baseline)
summary: 언어·회사 무관 공통 프로세스 baseline — 브랜치 보호, 커밋 타입 컨벤션, PR 규칙·체크리스트, 문서 위치, 테스트 요건, 태스크 러너 관례. 사내 룰이 있으면 사내 룰이 우선한다.
created: 2026-06-18
updated: 2026-08-19
visibility: team
scope: company
applies_to: [git, pr, process]
tags: [git, pr, process, team, rules, baseline]
source_file: 이전 팀 룰(Git/Doc/TDD/태스크러너 섹션)에서 조직 고유 항목을 제거해 일반화
---

# Git/PR + 공통 프로세스 규칙 (baseline)

> **성격**: 특정 회사의 규칙이 아니라 **어느 조직에서도 출발점이 되는 baseline**이다.
> 사내에 확정된 규칙이 있으면 **사내 규칙이 우선**한다. 이 문서는 사내 규칙이 없거나
> 명시되지 않은 영역을 메우는 기본값으로 쓴다.
>
> 조직마다 다른 값(이슈 프리픽스, base 브랜치명, 포매터, 태스크 러너)은 본문에서
> `{중괄호}` 자리표시자로 두었다. 실제 값은 프로젝트 CLAUDE.md에서 확정한다.

## Git Rules

### Branch Protection

| Branch Type | Push Allowed |
|----------|--------------|
| 기본 브랜치 (`main`) | **FORBIDDEN** |
| 통합 브랜치 (`dev`/`develop`) | **FORBIDDEN** |
| 배포 브랜치 (`release`, `stg`) | **FORBIDDEN** |
| `feature/*`, `fix/*`, `hotfix/*` | PR only |
| `conf/*`, `refactor/*`, `chore/*`, `docs/*` | PR only |

**Before push**: `git branch --show-current` — 보호 브랜치에 직접 push하지 않는다.

### Commit Message

**Format**: `type: 내용`

| Type | Description |
|------|-------------|
| `feature` | 새로운 기능 |
| `fix` | 버그 수정 |
| `refactor` | 리팩토링 |
| `docs` | 문서 작업 |
| `conf` | 설정 파일 변경 |
| `test` | 테스트 코드 |
| `style` | 코드 스타일 (로직 변경 없음) |
| `chore` | 빌드, 기타 작업 |
| `hotfix` | 긴급 장애 대응 |

**Examples**:
```
feature: 사용자 인증 기능 추가
fix: 널 참조 오류 수정
refactor: UserService 로직 개선
```

> 조직이 Conventional Commits(`feat:`/`fix:`/`chore:`)를 쓰면 그쪽을 따른다.
> 중요한 건 표기법이 아니라 **한 저장소 안에서 하나로 통일**하는 것이다.

### PR Rules

**Title Format**: `[{ISSUE-KEY}] type: description`

- `{ISSUE-KEY}`는 사내 이슈 트래커의 키를 쓴다 (예: `ABC-123`). 트래커가 없으면 생략한다.
- Base branch: 통합 브랜치(`{dev}`). hotfix만 예외.
- **PR 하나에 태스크 하나.** 기능 혼합 금지 — 리뷰 난이도와 롤백 단위가 함께 커진다.
- 불필요한 커밋은 정리하고(squash) push한다.
- **PR 전에 base 브랜치로부터 rebase**한다: `git fetch origin && git rebase origin/{dev} && git push --force-with-lease`
- **PR 본문에 기획/설계 문서 링크를 첨부**한다(스펙 md 경로, 이슈 URL 등). 리뷰어와 미래의 담당자가 작업 근거를 따라갈 수 있어야 한다.

**Checklist**:
- [ ] 테스트 코드 작성 완료
- [ ] 로컬 빌드 통과
- [ ] base 브랜치 rebase 완료
- [ ] 포매터/린터 검증 통과
- [ ] 기획·설계 문서 링크 첨부

---

## Document Location

| Location | Allowed |
|----------|---------|
| `{project}/docs/` | **O** |
| `{submodule}/docs/` | **X** — 서브모듈은 상위 프로젝트와 생명주기가 달라 문서가 흩어진다 |

**Subfolders**: `backend/`, `database/`, `frontend/`, `plans/`, `test/`, `qa/`, `infra/`, `guides/`

---

## 테스트 요건

- **비즈니스 로직·유틸리티 성격의 코드는 테스트 코드 필수.** 언어와 무관하다.
- 테스트 미작성 시 PR 리뷰 반려.
- 제외 대상(단순 DTO, 자동 생성 코드, 설정 파일)은 프로젝트 CLAUDE.md에 명시적으로 적는다. 적지 않으면 "테스트 안 써도 되는 코드"의 범위가 사람마다 달라진다.

## 태스크 러너

반복 명령은 셸 히스토리에 두지 말고 **태스크 러너 파일 하나로 모은다**(`justfile`·`Makefile`·
`package.json` scripts 등 — 프로젝트가 쓰는 것). 신규 합류자가 그 파일만 보면 되는 상태를 유지한다.

최소한 다음이 한 줄 명령으로 있어야 한다.

| 목적 | 예 |
|---|---|
| 포매터 검증 / 자동 수정 | `{run} format-check`, `{run} format` |
| Git hooks 설치 | `{run} install-hooks` |
| 로컬 개발 서버 기동 | `{run} dev` |
| DB 마이그레이션 상태 확인 / 적용 | `{run} migrate-status`, `{run} migrate` |

## 배경/이유
- 조직 고유 값(이슈 프리픽스·특정 포매터·특정 마이그레이션 도구)을 본문에서 걷어내고 자리표시자로 바꿔, 이직·신규 프로젝트에서 그대로 재사용 가능한 baseline으로 만들었다(2026-08-19).
- 사내 실제 규칙은 입사 후 `wiki-harvest`로 수확해 별도 파일로 둔다. 이 baseline과 충돌하면 사내 규칙이 이긴다.
- 관련: [[rules/stacks/nestjs]] · [[rules/stacks/nestjs-test]]
