---
type: source
title: Squad — Multi-AI agent terminal collaboration tool
summary: Claude Code·Codex CLI·Gemini CLI·OpenCode를 로컬 SQLite 메시지와 역할 기반 작업으로 연결하는 오픈소스 터미널 오케스트레이터의 공식 README 요약.
created: 2026-07-13
updated: 2026-07-13
visibility: private
url: https://github.com/mco-org/squad
author: mco-org
ingested_via: web
tags: [squad, ai-agent, orchestrator, claude-code, codex]
---

## 요약

Squad는 여러 AI CLI가 각자 터미널에서 실행되면서 로컬 SQLite를 통해 메시지와 구조화된 작업을 교환하게 한다. 별도 daemon이나 background server 없이 일회성 CLI 명령으로 동작하며 Claude Code, Gemini CLI, Codex CLI, OpenCode를 지원한다.

기본 역할은 manager, worker, inspector이다. manager는 목표를 분해하고 작업을 할당하며, worker는 작업을 실행하고, inspector는 결과를 검토한다. 사용자 정의 역할과 team YAML도 추가할 수 있다.

## 확인한 설치·동작

- macOS 설치: `brew install mco-org/tap/squad`
- 연동 설치: `squad setup` 또는 플랫폼별 `squad setup claude`, `squad setup codex`
- 프로젝트 초기화: `squad init`
- 상태 저장: 프로젝트의 `.squad/messages.db`
- 기본 역할: manager, worker, inspector
- 라이선스: MIT
- 확인 버전: 0.7.6 (2026-07-13 로컬 설치 및 공식 release 확인)

## 주의점

- `squad init`은 `.squad/`를 만들고 `.gitignore`, `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`에 협업 안내를 추가할 수 있다.
- 여러 에이전트가 같은 working tree를 수정하면 충돌할 수 있으므로 작업 경계 또는 Git worktree 격리가 필요하다.
- `squad clean`은 로컬 협업 상태를 지우므로 사용자 확인 없이 실행하면 안 된다.

## 추출한 영구노트

- [[agentic-workflow-patterns]] — Squad의 manager/worker/inspector 구조는 Orchestrator-Workers와 Evaluator-Optimizer의 실용적 조합으로 볼 수 있다.

## 출처 원문 메모

- 공식 README: https://github.com/mco-org/squad
- 공식 release: https://github.com/mco-org/squad/releases/tag/v0.7.6
