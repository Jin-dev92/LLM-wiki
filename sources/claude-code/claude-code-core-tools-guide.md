---
type: source
title: Claude Code 핵심 도구(플러그인·스킬) 설치 가이드
summary: 실무 추천 Claude Code 도구 6종(Superpowers·GStack·Matt Pocock·Codex Review·Vercel React·PPTX/PDF)의 When/Why/How 설치표와 Matt Pocock 스킬 상세 활용성.
created: 2026-06-18
updated: 2026-06-18
visibility: private
url: 
author: 본인 (iCloud claude/guides/ai)
ingested_via: pdf
tags: [claude-code, onboarding, skills, plugins, gstack, superpowers]
---

## 추출한 영구노트
- [[claude-code-setup]]

## 출처 원문 (iCloud 사본 — 2026-06-18 동기화)

# Claude Code 핵심 도구(플러그인 & 스킬) 설치 및 활용 가이드

Claude Code를 효율적으로 사용하기 위해 실무 개발자가 추천하는 6가지 핵심 도구의 사용 시점, 목적, 그리고 설치 방법을 정리한 문서입니다.

## 🛠 도구별 상세 안내

| 도구 이름 | 사용 시점 (When) | 사용 목적 (Why) | 설치 및 설정 방법 (How) |
| :--- | :--- | :--- | :--- |
| **Superpowers** | 새로운 기능 개발 시작 전 또는 대규모 작업 할당 시 | 성급한 코딩을 방지하고 단계별 워크플로우(기획->테스트->코드->리뷰)를 강제하여 정확도 향상 | 1. 마켓플레이스 등록:<br>`/plugin marketplace add obra/superpowers-marketplace`<br>2. 플러그인 설치:<br>`/plugin install superpowers@superpowers-marketplace` |
| **GStack** | 아이디어 구체화 단계 또는 빠른 웹 QA가 필요할 때 | **office-hour:** YC 방식의 질문으로 사업/기능 아이디어를 검증<br>**browse:** 백그라운드 세션으로 기존 대비 20배 빠른 웹 브라우징 수행 | 1. 저장소 클론 및 설정:<br>`git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack`<br>`cd ~/.claude/skills/gstack && ./setup` |
| **Matt Pocock's Skills** | PR(Pull Request) 전 코드 리뷰 혹은 아키텍처 개선 시 | **grill-me:** 비판적 동료로서 코드의 허점 지적<br>**improve-codebase-architecture:** 테스트 용이성과 재사용성을 고려한 구조 개선 제안 | 1. 스킬 설치:<br>`npx skills@latest add mattpocock/skills`<br>2. 에이전트 내 설정 시작:<br>`/setup-matt-pocock-skills`<br>→ [상세 내용](#matt-pocock-skills-상세) |
| **Codex Review** | 최종 병합(Merge) 직전 최종 검증 단계 | 클로드가 놓칠 수 있는 세부 오류를 OpenAI Codex 모델의 시각으로 교차 검증 | 1. 마켓플레이스 등록 및 설치:<br>`/plugin marketplace add openai/codex-plugin-cc`<br>`/plugin install codex@openai-codex`<br>2. 설정 실행: `/codex:setup` |
| **Vercel React Best Practices** | 리액트 및 Next.js 프로젝트 진행 시 | Vercel 팀의 최신 성능 최적화 사례와 서버/클라이언트 컴포넌트 활용 패턴을 강제 | 1. 스킬 설치:<br>`npx skills add vercel-labs/agent-skills`<br>(또는 특정 스킬만 지정: `--skill vercel-react-best-practices`) |
| **PPTX / PDF Skills** | 클라이언트 보고서나 발표 자료 작성이 필요할 때 | 개발자가 작성하기 번거로운 슬라이드나 문서 형태의 결과물을 자동 생성 | 1. 스킬 설치:<br>`npx skills add skills-ai/presentation-tools`<br>(공식 스킬샵 또는 `skills.ai` 참고) |

---

---

## Matt Pocock Skills 상세

> 출처: [yscho03.tistory.com/402](https://yscho03.tistory.com/402) · 설치일: 2026-05-17

Total TypeScript 창시자 Matt Pocock이 공개한 실무 AI 워크플로우 스킬 모음. MIT 라이선스로 자유롭게 사용 가능.

### 설치 명령

```bash
npx skills@latest add mattpocock/skills
```

설치 위치: `{프로젝트}/.agents/skills/` → Claude Code에 자동 심링크

### 스킬 목록 및 한국 개발자 활용성

| 스킬 | 사용 가능 여부 | 설명 |
|---|:---:|---|
| `tdd` | O | Red-Green-Refactor 루프 자동화. "TDD로", "테스트 먼저" 트리거 |
| `diagnose` | O | 버그/성능 회귀를 6단계(재현→가설→계측→수정→회귀테스트→사후분석)로 해결 |
| `grill-me` | O | 결정 전 모든 허점을 철저히 인터뷰 — 잘못된 방향 조기 차단 |
| `caveman` | O | 토큰 75% 절감하는 초압축 커뮤니케이션 모드 |
| `triage` | O | 이슈/버그 분류 워크플로우 |
| `zoom-out` | O | 디테일에 빠졌을 때 전체 그림 복귀 |
| `write-a-skill` | O | 새 스킬 작성용 스킬 |
| `prototype` | O | 빠른 프로토타이핑 |
| `handoff` | O | 작업 인계/컨텍스트 전달 |
| `to-issues` | △ | 대화→GitHub/Jira 이슈 변환. 본인 트래커에 맞게 수정 필요 |
| `to-prd` | △ | 대화→PRD 변환. 한국 회사 양식과 다를 수 있어 템플릿 수정 권장 |
| `improve-codebase-architecture` | △ | 아키텍처 개선. 한국어 코멘트/네이밍 컨벤션 반영 시 더 정확 |
| `grill-with-docs` | △ | 문서 업데이트 동반 플래닝. 사내 문서 시스템(Notion, Confluence) 맞게 손봐야 함 |
| `setup-matt-pocock-skills` | △ | 초기 셋업. 한국 환경(Notion, Slack 등)으로 답하면 됨 |
| ~~`migrate-to-shoehorn`~~ | X | Matt 본인 라이브러리 전용 — 미설치 |
| ~~`scaffold-exercises`~~ | X | 강의용 연습문제 생성 — 미설치 |

> O: 그대로 사용 가능 / △: 수정 권장 / X: 불필요(미설치)

### 주목할 스킬

**`grill-me`** — 결정 전에 AI가 12개 안팎의 날카로운 질문을 던짐. 캐시 무효화 기준, 엣지 케이스 등 놓치기 쉬운 부분을 잡아낸다.

**`tdd`** — 핵심 안티패턴 경고: "테스트 다 쓰고 구현 다 하는 horizontal slicing 금지". 기능 단위로 Red-Green-Refactor 루프를 돌려야 함.

**`caveman`** — 장시간 컨텍스트가 쌓였을 때 응답이 길어지는 문제를 해결. 의사결정 속도 향상에 효과적.

---

## 💡 활용 팁
* **단계별 도입:** 모든 도구를 한꺼번에 설치하면 컨텍스트가 너무 무거워질 수 있습니다. 자신의 현재 워크플로우에서 가장 필요한 도구부터 하나씩 추가하세요.
* **상태 확인:** 설치 후 Claude Code 내에서 `/help` 명령어를 입력하여 각 도구의 명령어가 정상적으로 등록되었는지 확인하세요.
* **차분한 워크플로우:** 이 도구들의 핵심은 클로드가 바로 코드를 뱉지 않게 '속도를 늦추는 것'입니다. 질문이 많아지더라도 정확한 결과를 위해 과정을 즐기세요.

## 🔗 참고 자료
* **원본 영상:** [400시간 만에 알게 된 Claude Code의 진짜 핵심 도구 6개](http://www.youtube.com/watch?v=BZPaZzjLIOY)
* **스킬 샵:** [https://skills.ai](https://skills.ai) (영상 제작자 운영 사이트)
