---
type: project
title: Squad로 Codex와 Claude Code 오케스트레이션하기
summary: macOS에서 Squad를 설치하고 Codex manager·Claude Code worker·Codex inspector 구조로 개발 작업을 운영하는 복원용 가이드. 프로젝트 초기화, 첫 실행, 충돌 방지, 복구, 제거 절차를 포함한다.
created: 2026-07-13
updated: 2026-07-13
visibility: private
status: active
tags: [squad, codex, claude-code, ai-orchestration, onboarding, setup]
---

# Squad로 Codex와 Claude Code 오케스트레이션하기

Squad를 이용해 Codex와 Claude Code를 역할별로 연결하는 방법을 정리한 글입니다. AI CLI를 여러 터미널에서 동시에 실행하면 작업 전달과 상태 추적이 사람의 기억에 의존하기 쉽습니다. Squad는 이 사이에 로컬 작업 큐와 메시지 채널을 둡니다.

> 작성 근거: [[sources/llm-agents/squad-multi-ai-orchestrator]], 로컬 Squad 0.7.6 CLI와 생성된 연동 파일. 작성 계획은 `docs/plans/2026-07-13-squad-installation-guide.md`에 있습니다.

## TL;DR

- `brew install mco-org/tap/squad` 후 Claude Code와 Codex 연동을 각각 설치합니다.
- Codex는 manager·inspector, Claude Code는 worker로 실행합니다.
- 시작 전 Git 상태를 깨끗하게 만들고 에이전트별 수정 경계를 명시합니다.

## 1. Squad가 해결하는 문제

Squad는 여러 AI CLI가 로컬 SQLite를 통해 메시지와 구조화된 작업을 주고받게 하는 터미널 협업 도구입니다. **Orchestrator-Workers**는 중앙 역할이 작업을 나누고 worker가 실행하는 패턴이며, **Evaluator-Optimizer**는 구현과 평가를 분리해 결과를 개선하는 패턴입니다. Squad의 manager·worker·inspector는 두 패턴을 함께 적용하기 좋습니다. 자세한 개념은 [[agentic-workflow-patterns]]를 참고합니다.

이 가이드에서는 “로그인 API에 사용자 ID와 IP 기반 rate limit을 추가한다”라는 작업을 끝까지 같은 예시로 사용합니다.

```text
사용자
  │ 목표·제약·성공 기준
  ▼
Codex manager ── 작업 할당 ──▶ Claude Code worker
  ▲                                 │
  │                                 │ 구현 결과
  │                                 ▼
  └── 재작업 요청 ─────────── Codex inspector

공유 상태: 프로젝트의 .squad/messages.db
```

1. 사용자가 manager에게 목표와 금지 사항을 전달합니다.
2. manager가 작업을 분해하고 worker에게 구조화된 task를 할당합니다.
3. worker가 허용된 범위에서 구현하고 테스트 결과를 보고합니다.
4. inspector가 diff와 테스트를 검토해 PASS 또는 수정 요청을 보냅니다.
5. 사용자가 최종 diff를 승인한 뒤 commit·push·PR을 별도로 진행합니다.

Squad는 작업 전달을 관리하지만 코드 품질이나 권한 경계를 자동 보장하지는 않습니다. `AGENTS.md`, `CLAUDE.md`, 테스트와 Git 검토가 계속 필요합니다.

## 2. 선택지 비교

| 방법 | 장점 | 단점 | 적합한 상황 |
|---|---|---|---|
| 단일 Codex 또는 Claude Code | 단순하고 컨텍스트 전달이 없음 | 구현과 검토가 같은 컨텍스트에 묶임 | 작은 수정 |
| Claude Code subagent/Agent Teams | Claude 안에서 구성이 간단함 | Codex를 역할에 포함할 수 없음 | Claude 중심 병렬 작업 |
| Squad | 서로 다른 AI CLI와 역할을 연결함 | 여러 터미널과 작업 경계 관리가 필요함 | Codex 기획·검토 + Claude 구현 |
| LangGraph 같은 API 오케스트레이터 | 상태·승인·분기를 코드로 정밀 제어 | 직접 개발과 별도 API 비용이 필요함 | 서비스 수준 자동화 |

현재 목적에는 Squad가 가장 작은 구성입니다. 모델 호출은 각 AI CLI가 담당하고 Squad는 역할, 메시지, 작업 상태만 연결합니다.

## 3. 사전 준비

macOS 기준으로 다음 도구가 필요합니다.

```bash
brew --version
claude --version
codex --version
git --version
```

Claude Code와 Codex는 각각 먼저 로그인되어 있어야 합니다. Squad 설정이나 `.squad/`에 모델 API 키를 기록하지 않습니다.

## 4. 전역 설치와 AI CLI 연동

### 4.1 Squad 설치

```bash
brew install mco-org/tap/squad
squad --version
```

이 문서는 2026-07-13에 Squad 0.7.6으로 검증했습니다. 복원 시에는 설치된 버전을 직접 확인하고, 버전 차이가 크면 공식 README와 `squad --help`를 다시 비교합니다.

### 4.2 지원 AI CLI 확인

```bash
squad setup --list
```

현재 환경에서 필요한 플랫폼만 설치합니다. 전체 자동 감지 설치보다 변경 범위가 명확합니다.

```bash
squad setup claude
squad setup codex
```

생성 경로는 다음과 같습니다.

```text
~/.claude/commands/squad.md
~/.codex/skills/squad/SKILL.md
```

새 명령과 스킬을 확실히 로드하려면 실행 중인 Claude Code와 Codex 세션을 종료하고 새 세션을 엽니다.

## 5. 프로젝트 초기화

`squad init`은 프로젝트 파일을 변경할 수 있습니다. 먼저 기존 변경을 확인합니다.

```bash
cd /path/to/project
git status --short
squad init
git diff -- .gitignore AGENTS.md CLAUDE.md GEMINI.md
```

초기화 후 주로 다음 항목이 생깁니다.

```text
.squad/
├── messages.db
├── roles/
└── teams/
```

Squad는 `.squad/`를 `.gitignore`에 추가하고, 누락된 경우 `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`에 협업 안내를 추가할 수 있습니다. 기존 프로젝트 규칙과 충돌하지 않는지 diff를 직접 검토합니다.

내장 역할 파일만 최신 기본값으로 갱신할 때는 다음 명령을 사용합니다.

```bash
squad init --refresh-roles
```

사용자 정의 role은 자동으로 덮어쓰지 않지만, 실행 전 diff와 백업 여부를 확인하는 편이 안전합니다.

## 6. 첫 팀 실행

같은 프로젝트 경로에서 터미널을 세 개 엽니다.

### 6.1 터미널 1 — Codex manager

Codex 새 세션에서 다음을 입력합니다.

```text
$squad manager
```

manager에게 작업 목표를 전달합니다.

```text
로그인 API에 백엔드 rate limit을 추가해줘.
사용자 ID와 IP를 함께 제한하고 프론트엔드만의 제한은 금지해.
Claude worker가 구현하고 Codex inspector가 diff와 테스트를 검토하게 해.
파일 삭제, push, PR 생성, merge, deploy는 하지 마.
```

### 6.2 터미널 2 — Claude Code worker

Claude Code 새 세션에서 다음을 입력합니다.

```text
/squad worker
```

worker는 manager가 보낸 task를 수신하고 구현합니다. 같은 역할을 여러 개 실행할 때는 고유 ID를 붙입니다.

```text
/squad worker worker-2
```

### 6.3 터미널 3 — Codex inspector

Codex 새 세션에서 다음을 입력합니다.

```text
$squad inspector
```

inspector에는 다음 검토 기준을 추가로 전달합니다.

```text
구현 계획, git diff, 테스트 결과만 근거로 독립 리뷰해.
백엔드의 사용자 ID+IP 이중 제한, API 키 비노출, RBAC을 확인해.
Supabase/PostgreSQL 변경이 있다면 RLS 우회 경로와 구독·사용량 테이블 분리를 확인해.
문제가 있으면 PASS 대신 수정 파일과 재현 방법을 worker에게 보내.
```

각 `$squad`·`/squad` 명령은 workspace 초기화, agent 등록, 역할 지침 로딩, 메시지 대기 흐름을 수행합니다.

## 7. 진행 상태 확인과 수동 제어

일상적으로 필요한 명령은 다음과 같습니다.

```bash
# 접속한 agent와 client 확인
squad agents

# task 상태 확인
squad task list

# 읽지 않은 메시지 확인
squad pending

# 전체 메시지 이력 확인
squad history

# 호환성 진단 — 상태를 변경하지 않음
squad doctor
```

CLI에서 직접 task를 만들 수도 있습니다.

```bash
squad task create manager worker \
  --title "로그인 API rate limit" \
  --body "사용자 ID+IP 이중 제한을 구현하고 테스트 결과를 보고한다"
```

task는 `create → ack → complete` 상태로 추적합니다. 단순 메모는 `squad send`, 완료 여부를 추적해야 하는 작업은 `squad task`를 사용합니다.

## 8. 충돌을 막는 운영 규칙

### 8.1 같은 working tree를 사용할 때

작업 전 다음 조건을 지킵니다.

1. `git status --short`가 비어 있는지 확인합니다.
2. manager만 작업을 분배합니다.
3. worker별 `allowed_paths`를 겹치지 않게 정합니다.
4. inspector는 기본적으로 파일을 수정하지 않고 리뷰만 합니다.
5. commit·push·PR·merge·deploy는 사람 승인 뒤 별도 단계로 진행합니다.

### 8.2 같은 파일을 수정해야 할 때

Git worktree로 worker를 격리하거나 한 worker가 끝난 뒤 다음 worker를 순차 실행합니다. Squad의 메시지 전달은 Git 충돌을 해결하지 않습니다.

### 8.3 비밀값과 보안 규칙

- API 키, access token, `.env` 내용은 task 본문이나 Squad 메시지에 넣지 않습니다.
- 민감한 외부 API는 프론트엔드에서 직접 호출하지 않습니다.
- `NEXT_PUBLIC_`, `VITE_` 같은 클라이언트 노출 prefix에 비밀키를 넣지 않습니다.
- rate limit은 백엔드에서 사용자 ID와 IP를 함께 제한합니다.
- Supabase/PostgreSQL 테이블은 RLS와 우회 경로를 검토하고 RBAC을 구현합니다.
- 구독 상태와 사용량 제한 데이터는 별도 테이블로 분리합니다.
- 클라우드 API를 사용하면 공급자 예산 알림을 코드 외부에서 설정합니다.

## 9. 중단 후 복구

먼저 상태를 확인합니다.

```bash
squad agents --all
squad task list
squad pending
squad history
```

`Session replaced`가 나오면 같은 ID를 다른 터미널이 사용한 것입니다. 고유 ID로 다시 참여합니다.

```bash
squad join worker-2 --role worker --client claude --protocol-version 2
```

모든 agent가 stale 상태일 때만 초기화를 고려합니다. `squad clean`은 프로젝트의 Squad 상태를 지우므로 먼저 사용자 확인을 받고 실행합니다.

```bash
squad clean
squad init
```

## 10. 업데이트와 제거

### 업데이트

```bash
brew update
brew upgrade mco-org/tap/squad
squad --version
squad setup claude
squad setup codex
```

업데이트 후 새 세션에서 manager·worker·inspector가 정상 등록되는지 작은 문서 전용 작업으로 확인합니다.

### 제거

전역 slash command와 Codex skill을 먼저 제거한 뒤 Homebrew 패키지를 삭제합니다.

```bash
squad cleanup
brew uninstall squad
```

`squad cleanup`은 AI 도구의 전역 연동을 제거하고, `squad clean`은 현재 프로젝트의 협업 상태를 지웁니다. 두 명령은 목적이 다릅니다. 프로젝트의 `.squad/`와 기존 대화·작업 이력은 개인 데이터이므로 삭제 전에 별도로 확인합니다.

## 11. 복원 체크리스트

- [ ] `squad --version`이 출력됨
- [ ] `squad setup --list`에서 Claude Code와 Codex가 감지됨
- [ ] Claude Code에서 `/squad worker`를 인식함
- [ ] Codex에서 `$squad manager`, `$squad inspector`를 인식함
- [ ] `squad agents`에서 각 client와 고유 ID가 확인됨
- [ ] manager가 task를 만들고 worker가 완료 상태를 보고함
- [ ] inspector가 별도 컨텍스트에서 diff와 테스트를 검토함
- [ ] `.squad/`, API 키, 사용자 비밀값이 Git에 포함되지 않음
- [ ] push·PR·merge·deploy 전에 사람 승인 단계가 있음

## 마치며

Squad는 Codex와 Claude Code 사이의 역할 분담과 작업 전달을 작게 시작할 수 있게 합니다. 다만 에이전트가 수정할 파일 범위, 검증 명령, 외부 변경 승인까지 대신 결정하지는 않습니다. 작은 작업에서는 단일 agent를 사용하고, 구현과 독립 리뷰를 분리할 가치가 있을 때 manager·worker·inspector 구성을 적용하는 편이 운영 비용을 줄입니다.
