---
type: note
title: Spec-as-Code (테스트 코드 = 살아있는 명세)
summary: E2E 테스트 코드는 읽으면 사용자 명세, 돌리면 동작 검증인 "두 얼굴"이다. 명세가 코드와 한 몸이라 코드가 바뀌면 명세도 같이 바뀌어 문서-코드 불일치가 사라진다.
created: 2026-06-22
updated: 2026-06-22
visibility: private
provenance: extracted
tags: [testing, e2e, spec, documentation, agent]
---

## 핵심
사용자 수준에서 작성한 E2E 테스트 코드는 **읽으면 명세(spec), 돌리면 검증(verification)**이라는 두 얼굴을 가진다. 명세 문서와 실제 동작이 *같은 한 벌*이므로, 코드가 바뀌면 명세도 함께 바뀌어 "무엇이 최신인지" 혼란이 사라진다.

## 상세
- **예시**: `page.goto('/loan')` → `getByRole('button',{name:'한도조회'}).click()` → `expect(getByText('한도조회 결과')).toBeVisible()` 코드는, 다르게 읽으면 *"사용자가 /loan에서 한도조회 버튼을 누르면 한도조회 결과가 보인다"*는 명세다.
- **성립 조건**: 함수가 아니라 **실제 사용자 경험**을 검증해야 한다(페이지 이동·요소 표시·클릭). 그래서 단위 테스트(함수 검증)가 아니라 E2E([[playwright-e2e-for-agents]]) 레벨에서 성립한다.
- **에이전트 관점의 가치**: 같은 한 벌이 [[self-improvement-loop]]의 가이드(명세를 읽혀 방향 제공)와 센서(돌려서 동작 검증)를 동시에 충족한다 → 별도의 명세 문서를 따로 유지·동기화할 필요가 없다.
- 시멘틱 셀렉터(`getByRole`/name)를 쓰면 코드가 더 "명세처럼" 읽히고 디자인 변경에 덜 깨진다.

## 관련
- [[playwright-e2e-for-agents]] — 이 명세를 실행하는 도구·기법
- [[self-improvement-loop]] — 가이드이자 센서로 쓰임
- [[harness-engineering]]
- 출처: [[sources/llm-agents/playwright-e2e-harness-naverpay]]
