---
type: note
title: AI 활용 경험을 이력서에 쓰는 법 (도구 사용 → 프로세스 설계·통제)
summary: 이력서에 "AI를 썼다"가 아니라 "AI가 안전하게 일할 개발 환경을 설계·통제했다"를 써라. plan-first+승인 게이트, 역할 분리, 로깅 3가지 신호와 복붙용 문구.
created: 2026-06-24
updated: 2026-06-24
visibility: private
provenance: extracted
tags: [career, resume, ai, llm, agent, harness, job-search]
---

## 핵심
이력서에서 AI 경험의 변별력은 **"무엇을 했다"(도구 사용)가 아니라 "AI가 안전하게 일하도록 프로세스를 어떻게 설계·통제했는가"**에서 나온다. `"Claude로 개발"`, `"Codex로 생산성 향상"` 같은 도구 언급은 이제 차별점이 아니며(누구나 쓰므로) 오히려 마이너스가 될 수 있다. 면접에서 풀어낼 수 있는 한 줄이 되려면 **통제 지점 + 품질/검증 메커니즘**을 드러내야 한다.

## 상세
도구를 그냥 썼다 → 프로세스를 설계했다로 프레이밍을 바꾸는 3가지 신호. 셋 다 [[harness-engineering]]의 구체적 적용(모델 외부 환경 설계)이며, 토이 프로젝트 하나로도 만들 수 있다.

### 1. plan-first + human approval gate
바로 구현시키면 AI가 의도 밖 파일·기존 API 응답 포맷까지 건드린다. "구현하지 말고 기존 구조 파악 후 작업 계획과 리스크만 정리" → **사람이 승인한 뒤** 구현으로 흐름을 통제. → [[workflow-vs-agent]]에서 Agent의 자율성을 사람 승인으로 제한하는 형태.
- 문구: **"AI 코드 생성 시, 작업 범위 이탈 및 변경 리스크 방지를 위한 plan-first workflow와 human approval gate를 적용해 개발 안정성 향상"**

### 2. 역할 분리 (Researcher / Planner / Reviewer)
하나의 AI에 조사·구현·리뷰를 다 시키지 않는다. Planner는 계획만, Reviewer는 직접 고치지 않고 문제점만 지적한다(리뷰어가 고치면 리뷰가 아니라 또 하나의 구현이 됨 → 검증 독립성 상실). → [[agentic-workflow-patterns]]의 Orchestrator·Evaluator 분리, [[agentic-reasoning-design-patterns]]의 Multi-agent와 같은 발상.
- 문구: **"Researcher, Planner, Reviewer 역할을 분리한 AI 개발 워크플로우를 구성해 구현과 검증 과정을 독립적으로 운영"**

### 3. prompt·tool 로그로 재현성·추적성 확보
결과물만 있고 어떤 질문을 했는지, AI가 어떤 파일을 읽고 어떤 명령을 실행했는지 기록이 없으면 문제 발생 시 원인 추적이 불가능하다. 로깅은 기록용이 아니라 **AI 작업을 블랙박스로 두지 않고 재현성·디버깅 가능성·운영성을 확보**하는 장치. → [[self-improvement-loop]]가 돌려면 필요한 센서 데이터이기도 하다.
- 문구: **"Prompt 및 logging hook을 구축해 AI agent 작업의 재현성 및 오류 원인 추적 가능성 향상"**

### 적용 메모
- 세 문구는 [[resume-writing-principles]]의 "기술↔문제 연결"·"문제→대처" 원칙과 결합해, 실제 [[dev-profile-kim-uijin]] 프로젝트 경험에 매핑해야 면접에서 풀린다. 그대로 복붙만 하면 출처 영상이 지적한 "유행 따라 쓴 문장"이 된다.
- **실제 구현(tip 3)**: private repo `Jin-dev92/cc-audit-hooks` — Claude Code 프롬프트·도구 로깅 + 위험명령 차단 + 리포트를 훅으로 직접 구현(2026-06-24). 설치/복원은 [[audit-logging-setup]]. → "AI 시대 개발 프로세스를 직접 설계"한 경험 근거로 이력서에 쓸 수 있다.
- provenance: 이력서 문구 3종은 영상에서 그대로 제시된 것(extracted). "셋 다 harness-engineering의 적용"이라는 연결은 내 해석(inferred).

## 관련
- [[resume-writing-principles]] — 상위 이력서 작성 원칙(이 노트는 AI 항목에 특화된 적용)
- [[harness-engineering]] — 3가지 신호의 공통 뿌리(모델 외부 환경 설계·통제)
- [[workflow-vs-agent]] / [[agentic-workflow-patterns]] / [[agentic-reasoning-design-patterns]] — 역할 분리·승인 게이트의 패턴 근거
- [[self-improvement-loop]] — 로깅이 공급하는 검증 회로
- [[dev-profile-kim-uijin]] — 문구를 매핑할 내 경력 프로필
- 출처: [[sources/career/ai-experience-on-resume-thinklighthouse]]
