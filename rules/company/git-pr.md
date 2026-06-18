---
type: rule
title: 팀 Git/PR + 공통 프로세스 규칙
summary: 언어 무관 공통 규칙 — Git 브랜치 보호, 커밋/PR 컨벤션(THUB-XXX), 문서 위치, TDD 요건, justfile 명령.
created: 2026-06-18
updated: 2026-06-18
visibility: team
scope: company
applies_to: [git, pr, process]
tags: [git, pr, process, team, rules]
source_file: iCloud/claude/docs/team-java-rules.md (Git/Doc/TDD/justfile 섹션)
---

# 팀 Git/PR + 공통 프로세스 규칙

## Git Rules

### Branch Protection

| Branch Type | Push Allowed |
|----------|--------------|
| `main` | **FORBIDDEN** |
| `dev` | **FORBIDDEN** |
| `release`, `stg` | **FORBIDDEN** |
| `feature/*`, `fix/*`, `hotfix/*` | PR only |
| `conf/*`, `refactor/*`, `chore/*`, `docs/*` | PR only |

**Before push**: `git branch --show-current`

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
fix: NPE 오류 수정
refactor: UserService 로직 개선
```

### PR Rules

**Title Format**: `[THUB-XXX] type: description`

- Base branch: always `dev` (except hotfix)
- Single task per PR (기능 혼합 금지)
- Squash commits before push (불필요한 커밋 정리)
- Rebase from dev before PR

**Checklist**:
- [ ] 테스트 코드 작성 완료
- [ ] 로컬 빌드 통과
- [ ] dev 브랜치 rebase 완료
- [ ] Spotless 검증 통과

---
## Document Location

| Location | Allowed |
|----------|---------|
| `{project}/docs/` | **O** |
| `{submodule}/docs/` | **X** (FORBIDDEN) |

**Subfolders**: `backend/`, `database/`, `frontend/`, `plans/`, `test/`, `qa/`, `infra/`, `inspect/`, `guides/`

---

## TDD Requirements

- Java 파일 변경 시 UTIL, SERVICE 성 파일 및 기능에 대해서는 반드시 **테스트 코드 필수**
- 테스트 미작성 시 PR 리뷰 실패

## justfile Commands

| Command | Description |
|---------|-------------|
| `just spotless-check` | 코드 스타일 검증 |
| `just spotless-fix` | 코드 스타일 자동 수정 |
| `just install-hooks` | Git Hooks 설치 |
| `just start-local` | React 개발서버 (local) |
| `just start-dev` | React 개발서버 (dev) |
| `just base-status [env]` | Liquibase 상태 확인 |
| `just base-update [env]` | Liquibase 마이그레이션 |
