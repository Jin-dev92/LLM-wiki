---
type: project
title: Claude Code 감사 로깅 훅 셋업 (cc-audit-hooks)
summary: Claude Code 프롬프트·도구 호출을 로깅하고 위험명령을 차단하는 cc-audit-hooks를 새 환경에서 복원하는 가이드 — clone → install.py → 검증 → 리포트.
created: 2026-06-24
updated: 2026-06-24
visibility: private
status: active
tags: [meta, claude-code, hooks, audit, logging, security, onboarding, setup]
---

## 목표
새 기기/계정에서 Claude Code **감사 로깅 훅**을 복원한다. 모든 프롬프트·도구 호출을
JSONL로 남기고, 위험한 Bash 명령을 실행 전 차단하며, 사용 현황을 Markdown 리포트로 뽑는다.

> 동기: 생각등대 "이력서에 AI 이렇게 씀" tip 3 — *"AI 작업을 블랙박스로 두지 말고 prompt·tool 로그로 재현성·추적성을 확보하라"*. 근거 노트 [[ai-experience-on-resume]], [[harness-engineering]].
> 소스 오브 트루스: **private repo `Jin-dev92/cc-audit-hooks`** (라이브 위치는 `~/.claude/hooks/audit/`).

---

## 1. 설치 (복원)
```sh
git clone https://github.com/Jin-dev92/cc-audit-hooks.git ~/orca/workspaces/cc-audit-hooks
cd ~/orca/workspaces/cc-audit-hooks
python3 install.py
```
- `install.py`가 하는 일(멱등):
  1. `hooks/audit/`(tests 제외)를 `~/.claude/hooks/audit/`로 복사
  2. `~/.claude/settings.json`의 `hooks`에 `UserPromptSubmit`/`PreToolUse`/`PostToolUse` 항목을 **기존 항목 보존하며** 병합 등록
- settings.json이 손상돼 있으면 복사 전에 멈춘다(부분 설치 방지). 재실행해도 중복 등록 안 됨.
- **새 세션부터 적용**(현재 세션에 즉시 적용될 수도 있음).

## 2. 검증
```sh
cd ~/orca/workspaces/cc-audit-hooks
python3 -m unittest discover -s hooks/audit/tests   # 29 tests, all pass
python3 -c "import json;print(list(json.load(open('$HOME/.claude/settings.json'))['hooks'].keys()))"
# -> ['PreToolUse', 'PostToolUse', 'UserPromptSubmit'] (기존 WebFetch 훅과 공존)
```

## 3. 사용
- 로그 위치(권한 0600): `~/.claude/logs/audit/YYYY-MM-DD.jsonl` + `index.db`(SQLite 인덱스).
- 리포트 생성: `python3 ~/.claude/hooks/audit/audit_report.py [out.md]`
  (기본 출력 `~/.claude/logs/audit/audit-report.md`) — 일자별 프롬프트 수·도구 빈도·**차단된 명령**·많이 건드린 파일.
- 차단 규칙 편집: `~/.claude/hooks/audit/danger_rules.json` (정규식 + action `deny`). 기본: rm 재귀+강제, force push, fork bomb, dd to device, mkfs, chmod -R 777, curl|sh.

## 4. 제거
```sh
rm -r ~/.claude/hooks/audit
# 그리고 ~/.claude/settings.json 의 UserPromptSubmit/PreToolUse/PostToolUse 에서
# audit_hook.py 항목을 수동 제거
```

---

## 설계/구현 메모
- 표준 라이브러리만(re/json/sqlite3) — pip 설치 0. 훅은 세션을 절대 깨지 않도록 모든 예외를 삼키고 exit 0(의도적 deny만 예외).
- PreToolUse deny 출력은 Claude Code 규약 `hookSpecificOutput.permissionDecision:"deny"` 사용.
- **레닥션은 휴리스틱**(sk-/ghp_/AKIA/Bearer/password= 등) — 100% 완전하지 않다. 로그 파일을 git에 올리거나 공유하지 말 것.
- 설계 스펙·구현 플랜은 repo의 `docs/specs/`, `docs/plans/`에 동봉.

## 주의 — 기존 SDD 훅과의 관계
이미 `~/.claude/settings.json`에는 [[claude-code-setup]] §5-1의 source-driven-development WebFetch 훅이 있다. cc-audit-hooks는 그와 **별도 항목으로 공존**(install.py가 기존 보존). 복원 순서는 무관.

## 동기화 규칙
- 원본: private repo `Jin-dev92/cc-audit-hooks`. 라이브: `~/.claude/hooks/audit/`.
- 규칙·코드를 바꾸면 repo에서 수정 후 `python3 install.py` 재실행으로 라이브 갱신.
