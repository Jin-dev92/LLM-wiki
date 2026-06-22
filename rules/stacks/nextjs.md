---
type: rule
title: Next.js(App Router) 개발 규칙
summary: Next.js App Router 기반 FE 규칙 — Server/Client Component 경계, 데이터 페칭/Server Action, NEXT_PUBLIC_ 민감키 금지, 내장 컴포넌트 우선, .ts/.tsx 강제.
created: 2026-06-22
updated: 2026-06-22
visibility: team
scope: stack
applies_to: [nextjs, react, typescript, app-router]
tags: [frontend, nextjs, react, typescript, rules]
---

# Next.js (App Router) Development Rules

> Next.js App Router + React Server Components 기반 FE 개발 규칙.
> Vite/CRA가 아니므로 `import.meta.env`·CSR 전용 패턴을 그대로 가져오지 않는다.

## 버전 주의

- 이 레포의 Next.js는 학습 데이터와 다를 수 있다. **API/관례/파일구조 변경 가능** —
  코드 작성 전 `node_modules/next/dist/docs/`의 해당 가이드를 먼저 읽고 deprecation을 확인한다.
- 버전별 세부 API(시그니처·옵션)는 **추측하지 말고 docs로 확인**한다.

## 파일/구조 규칙

- `.ts` / `.tsx`**만** 허용. `.js` / `.jsx` 신규 생성 금지 ❌
- 라우팅은 `app/` 디렉토리(App Router) 규칙을 따른다 — `page.tsx`, `layout.tsx`,
  `route.ts`(Route Handler), `loading.tsx`, `error.tsx` 등 파일 컨벤션 사용.
- 함수형 컴포넌트만 사용(클래스 컴포넌트 금지). → [[react]]

## Server / Client Component 경계

- **기본은 Server Component**. `"use client"`는 상호작용(상태·이벤트·브라우저 API)이
  실제로 필요한 컴포넌트에만 **최소 범위**로 선언한다.
- `"use client"`를 트리 상단에 무분별하게 붙이지 않는다 — 클라이언트 번들 비대화 방지.
- 클라이언트 전용 코드(브라우저 API, 이벤트 핸들러)는 Server Component에 두지 않는다.

## 데이터 페칭 / 뮤테이션

- 읽기 데이터는 **Server Component에서 `fetch`/직접 호출**로 가져오는 것을 우선한다.
  단순 조회에 클라이언트 페칭 라이브러리를 기본값으로 끌어들이지 않는다.
- 쓰기(뮤테이션)는 **Server Action** 또는 **Route Handler(`route.ts`)**를 통한다.
  정확한 시그니처/관례는 docs 확인 후 사용.
- 외부/백엔드 API 호출 로직을 컴포넌트 JSX 안에 인라인으로 흩뿌리지 않는다 —
  `lib/`(혹은 `app/_lib`) 등 데이터 레이어로 분리해 재사용한다.

## 환경변수 (보안)

- **`NEXT_PUBLIC_` prefix가 붙은 변수는 클라이언트 번들에 그대로 노출**된다.
  AI/Stripe/이메일/클라우드 등 **민감 키를 `NEXT_PUBLIC_`에 절대 넣지 않는다** ❌
- 민감 키는 prefix 없는 서버 전용 환경변수로 두고, Server Component / Route Handler /
  Server Action 등 **서버 측에서만** 사용한다.
- 민감한 외부 API는 클라이언트에서 직접 호출하지 않고 서버 측을 경유한다.

## 내장 기능 우선

- 이미지는 `next/image`, 내부 이동은 `next/link`, 폰트는 `next/font`를 사용한다
  (`<img>`/`<a>`/`<link rel=font>` 직접 사용 지양).
- `<head>` 직접 조작 대신 **Metadata API**(`metadata` export / `generateMetadata`)를 사용한다.

## TypeScript

- 공통 TS 규칙은 [[frontend]]의 "TypeScript 타입 규칙"을 따른다
  (`as any` 금지, index signature 금지, `enum`→`as const` 객체 패턴 등).
