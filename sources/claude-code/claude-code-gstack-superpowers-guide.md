---
type: source
title: Claude Code 워크플로우 가이드 (GStack + Superpowers)
summary: GStack+Superpowers 설치·검증·트러블슈팅과 둘의 역할 구분(GStack=무엇을 / Superpowers=어떻게) 중심 협업 워크플로우 가이드.
created: 2026-06-18
updated: 2026-06-18
visibility: private
url: 
author: 본인 (iCloud claude/guides/ai)
ingested_via: pdf
tags: [claude-code, onboarding, gstack, superpowers, workflow]
---

## 추출한 영구노트
- [[claude-code-setup]]

## 출처 원문 (iCloud 사본 — 2026-06-18 동기화)

# Claude Code 워크플로우 가이드: GStack + Superpowers

> **목적**: 단순 명령 대신 체계적인 협업 흐름으로 결과물 품질을 높이는 Claude Code 활용법

---

## 설치 가이드

### 사전 요구사항
- Claude Code 설치 및 실행 가능한 상태
- Git 2.5 이상 (Worktrees 사용 시)

### 1단계: Superpowers 설치

공식 Claude 플러그인 마켓플레이스를 통해 설치한다 (권장):

```bash
# Claude Code 내부에서 실행
/plugin install superpowers@claude-plugins-official
```

또는 커뮤니티 마켓플레이스를 통한 설치:

```bash
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

설치 후 **Claude Code를 재시작**하면 세션 시작 시 Superpowers가 자동으로 활성화된다.

### 2단계: GStack 설치

```bash
# 터미널에서 실행 (Claude Code 외부)
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup
```

설치 후 슬래시 커맨드 사용 가능 여부 확인:

```bash
# Claude Code 내부에서 확인
/office-hours   # GStack 설치 확인
/brainstorming  # Superpowers 설치 확인
```

### 설치 확인 & 문제 해결

| 증상 | 원인 | 해결 |
|------|------|------|
| `/office-hours` 미인식 | CLAUDE.md에 gstack 섹션 누락 | Claude Code 세션 재시작 |
| Superpowers 스킬 미작동 | 훅 스크립트 권한 문제 | `chmod +x ~/.claude/plugins/superpowers/hooks/session-start.sh` |
| Worktree 오류 | Git 버전 문제 또는 stale worktree | `git worktree prune` 후 재시도 |
| 캐시 문제로 재설치 필요 | - | `/plugin install superpowers@superpowers-marketplace --force` |

> **참고**: GStack과 Superpowers는 서로 역할이 다르므로 충돌하지 않는다.  
> GStack은 **무엇을** 만들지(역할 검토), Superpowers는 **어떻게** 만들지(구현 방식)를 담당한다.

---

## 왜 이 조합인가?

많은 플러그인이 있지만 기능이 너무 많아 복잡하거나 작업 흐름을 방해하는 경우가 많다.  
**GStack + Superpowers** 조합은 다음의 자연스러운 워크플로우를 제공한다:

```
아이디어 정리 → 계획 수립 → 분업 및 구현
```

---

## 주요 도구 및 기능

### 🔷 GStack — 프로젝트의 방향성 잡기

#### Office Hours (오피스 아워)
- 코딩 **전** Claude와 아이디어 회의를 하는 단계
- Claude가 질문을 던지며 불명확한 아이디어를 구체화
- 권장 시간: **15~20분**

```
활용 시점: 구현 시작 전, 아이디어가 막연할 때
```

#### Design Review (디자인 리뷰)
- 결과물에서 **AI Slop** 제거
  - 과도한 그라데이션
  - 불필요한 이모지
  - 전형적인 "AI스러운" 느낌
- 깔끔하고 전문적인 디자인 유지

```
활용 시점: 구현 완료 후, 디자인 검수 단계
```

---

### ⚡ Superpowers — 실행력과 조직력 강화

#### Sub-agent Driven Development (서브 에이전트)
- 큰 작업을 여러 개의 **작은 작업으로 분리**
- 각각의 전문 Claude(에이전트)에게 개별 할당
- 집중도 향상 → 품질 향상

```
활용 시점: 복잡하고 범위가 큰 작업을 시작할 때
```

#### Worktrees (워크트리)
- 작업 공간을 **독립적으로 분리**
- 프로젝트 간 간섭 차단
- 작업 내용 자동 기록

```
활용 시점: 여러 기능을 병렬로 개발할 때
```

#### Writing Plans (라이팅 플랜즈)
- 실제 구현 전 **단계별 설계 도면** 작성
- 중간에 방향을 잃지 않도록 가이드 역할

```
활용 시점: 구현 직전, 세부 계획이 필요할 때
```

---

## 추천 작업 순서 (Workflow)

```
Step 1  →  Step 2  →  Step 3  →  Step 4
```

| 단계 | 도구 | 작업 내용 | 소요 시간 |
|------|------|-----------|-----------|
| **1. 아이디어 상담** | GStack - Office Hours | 대화를 통해 기획 구체화 | 15~20분 |
| **2. 계획 수립 및 검토** | Superpowers - Brainstorming & Writing Plans | 단계별 계획서 작성 및 이중 검증 | - |
| **3. 공간 분리** | Superpowers - Worktrees | 작업 단위별 독립 공간 생성 | - |
| **4. 구현** | Superpowers - Sub-agent | 분리된 컨텍스트에서 에이전트 집중 개발 | - |

---

## 핵심 원칙

- ❌ `"이거 만들어줘"` — 맥락 없는 단순 명령 지양
- ✅ **Office Hours**로 방향 정렬 후 구현 시작
- ✅ **Writing Plans**로 중간 이탈 방지
- ✅ **Sub-agent**로 복잡도 분산
- ✅ **Design Review**로 AI Slop 제거

---

*참고: 이 가이드는 Claude Code GStack + Superpowers 플러그인 활용 영상 요약을 기반으로 작성되었습니다.*
