---
type: project
title: Squad 설치·사용 가이드 작성 계획
summary: Claude Code와 Codex CLI를 Squad로 연결하는 복원용 가이드의 범위, 근거, 검증 기준을 정의한다.
created: 2026-07-13
updated: 2026-07-13
visibility: private
status: completed
tags: [squad, ai-orchestration, documentation, plan]
---

# Squad 설치·사용 가이드 작성 계획

## 목표

새 macOS 환경이나 새 프로젝트에서 Squad를 설치하고, Codex를 manager·inspector로, Claude Code를 worker로 연결해 재현 가능한 개발 오케스트레이션을 시작할 수 있게 한다.

## 근거

- Squad 공식 저장소와 README
- 로컬에 설치된 Squad 0.7.6의 CLI 출력
- `squad setup`이 생성한 Codex·Claude Code 연동 파일
- [[agentic-workflow-patterns]]의 Orchestrator-Workers와 Evaluator-Optimizer 패턴

## 변경 범위

1. 공식 자료를 `sources/llm-agents/`에 요약한다.
2. `guide/ai/`에 설치·초기화·실행·트러블슈팅·제거 절차를 작성한다.
3. 하나의 인증 기능 예시로 manager → worker → inspector 흐름을 설명한다.
4. `index.md`에 source, guide, plan을 연결한다.

## 검증 기준

- 모든 새 문서에 필수 frontmatter가 있다.
- 설치 명령과 생성 경로가 Squad 0.7.6 및 공식 README와 일치한다.
- `squad init`이 프로젝트 파일을 수정한다는 경고가 있다.
- `squad clean`, push, PR, merge를 자동 실행하도록 안내하지 않는다.
- API 키나 사용자별 비밀값을 포함하지 않는다.
- Markdown 링크와 내부 WikiLink가 깨지지 않는다.
