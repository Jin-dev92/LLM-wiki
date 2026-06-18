---
type: rule
title: Frontend 개발 규칙 (React Query + Zustand + TS)
summary: React Query v5+Zustand+TypeScript+Axios 기반 FE 규칙. API 4파일 세트, QueryKey/Hook 패턴, Zustand vs RQ, 금지 패턴.
created: 2026-06-18
updated: 2026-06-18
visibility: team
scope: stack
applies_to: [react, typescript, react-query, zustand, axios]
tags: [frontend, react, typescript, rules]
source_file: iCloud/claude/docs/fe-rules.md
---

# Frontend Development Rules

> React Query v5 + Zustand + TypeScript(tsx) + Axios 기반 FE 개발 규칙.
> 레거시 패턴 없이 처음부터 모던하고 일관된 방식으로 개발하기 위한 가이드.

---

## 파일 구조 규칙

### 파일 확장자

- `.ts` / `.tsx` **만 허용**
- `.js` / `.jsx` **신규 생성 금지** ❌

### API 레이어 4파일 세트 의무화

새 도메인 작업 시 아래 4개 파일을 **반드시** 함께 생성한다.

```
src/api/{domain}.ts                              ← API 함수
src/types/api/{domain}.types.ts                  ← 타입 정의
src/hooks/queries/use{Domain}Queries.ts          ← Query hooks
src/hooks/mutations/use{Domain}Mutations.ts      ← Mutation hooks
```

각 디렉토리의 `index.ts`에 barrel export 추가 필수.

> 빠른 생성: `/add-api-domain {도메인명}` 스킬 사용

---

## 컴포넌트 규칙

- 컴포넌트 내 `axiosInstance` **직접 호출 금지** ❌
- 컴포넌트는 hook을 **소비**하기만 함 (API 호출 로직 포함 금지)

```typescript
// ❌ FORBIDDEN — 컴포넌트에서 직접 호출
const MyComponent = () => {
  const handleClick = async () => {
    const res = await axiosInstance.post("/api/orders", data); // ❌
  };
};

// ✅ GOOD — hook을 소비
const MyComponent = () => {
  const { mutate: createOrder } = useCreateOrder();
  const handleClick = () => createOrder(data);
};
```

---

## QueryKey 패턴

```typescript
// ✅ 올바른 패턴
export const orderQueryKeys = {
  all: ["order"] as const,
  list: () =>
    ["order", "/api/orders"] as const,
  byId: (orderId: string) =>
    ["order", "/api/order", orderId] as const,
  byCustomer: (customerId: number) =>
    ["order", "/api/order", customerId] as const,
};
```

**규칙:**
- `{domain}QueryKeys` 객체로 정의 (파일 상단)
- `all: ["domain"] as const` **필수** — 전체 무효화용
- 모든 key에 `as const` **필수**
- 첫 번째 요소는 항상 **도메인명 문자열**

---

## Query Hook 패턴

```typescript
// ✅ 올바른 패턴
export const useGetOrderById = (
  orderId: string | null | undefined,
  options?: Partial<UseQueryOptions<OrderResponse>>,
) =>
  useQuery<OrderResponse>({
    queryKey: orderQueryKeys.byId(orderId ?? ""),
    queryFn: () => getOrderById(orderId!).then((res) => res.data),
    enabled: !!orderId,
    ...options,
  });
```

**규칙:**
- 파라미터는 `string | null | undefined` 허용 (nullable 허용)
- 필수 파라미터 있을 때 `enabled`로 null 체크 **필수**
- `.then(res => res.data)`로 axios 응답에서 data만 추출

### staleTime 가이드

| 값 | 적용 대상 |
|---|---|
| `0` (default) | 실시간성 필요 데이터 |
| `1000 * 30` | 자주 변하지 않는 목록 |
| `1000 * 60 * 5` | 정적 코드 테이블 |

---

## Mutation Hook 패턴

```typescript
// ✅ 올바른 패턴 — onSuccess는 invalidate만
export const useCreateOrder = (
  options?: UseMutationOptions<OrderResponse, Error, CreateOrderRequest>,
) => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (req: CreateOrderRequest) =>
      createOrder(req).then((res) => res.data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: orderQueryKeys.all });
    },
    ...options,
  });
};
```

**규칙:**
- `onSuccess`는 `invalidateQueries`**만** — toast/alert/snackbar **금지** ❌
- UI 피드백은 소비 컴포넌트의 `onSuccess` 또는 `useEffect`에서 처리
- 이중 invalidation 금지 (hook + 컴포넌트 양쪽에서 동시에) ❌

```typescript
// ❌ FORBIDDEN — hook에서 UI 피드백
onSuccess: () => {
  queryClient.invalidateQueries({ queryKey: orderQueryKeys.all });
  enqueueSnackbar("완료"); // ❌ 금지
},

// ✅ GOOD — 소비 컴포넌트에서 UI 처리
const { mutate } = useCreateOrder({
  onSuccess: () => enqueueSnackbar("주문이 생성되었습니다."),
});
```

---

## React Query v5 패턴

### deprecated 패턴 사용 금지

`useQuery` / `useMutation` 옵션의 `onSuccess` / `onError` 콜백은 **v5에서 deprecated**.

```typescript
// ❌ FORBIDDEN — v5 deprecated
useQuery({
  queryKey: [...],
  queryFn: fetchData,
  onSuccess: (data) => { ... }, // ❌ deprecated
  onError: (err) => { ... },    // ❌ deprecated
});

// ✅ GOOD — useEffect로 처리하거나 컴포넌트 옵션으로 전달
const { data, isSuccess } = useGetOrders();
useEffect(() => {
  if (isSuccess) { ... }
}, [isSuccess]);
```

### isSaving 수동 boolean state 금지

```typescript
// ❌ FORBIDDEN
const [isSaving, setIsSaving] = useState(false);

// ✅ GOOD
const { mutate, isPending } = useCreateOrder();
```

---

## TypeScript 타입 규칙

### index signature 금지

```typescript
// ❌ FORBIDDEN — 필드가 명확히 정의된 경우
interface OrderFilter {
  [key: string]: any; // ❌
}

// ✅ GOOD — 명확한 필드 정의
interface OrderFilter {
  customerId?: number;
  status?: OrderStatus;
}
```

### `as any` 금지

```typescript
// ❌ FORBIDDEN
const data = response as any;

// 불가피한 경우 — 사유 주석 필수
const data = response as any; // TODO: API 응답 타입 정의 후 제거 예정
```

### API 함수 제네릭 타입 명시 필수

```typescript
// ✅ GOOD
axiosInstance.get<OrderResponse>("/api/orders");
axiosInstance.post<CreateOrderResponse, CreateOrderRequest>("/api/orders", req);

// ❌ FORBIDDEN — 제네릭 없음
axiosInstance.get("/api/orders");
```

### 타입 파일 작성

- OpenAPI 스펙 또는 실제 사용 코드 기반으로 작성
- 출처 주석 필수

```typescript
// @see OpenAPI spec: POST /api/orders
export interface CreateOrderRequest {
  customerId: number;
  items: OrderItem[];
}
```

---

## Store 규칙 (Zustand vs React Query)

| 상태 유형 | 도구 |
|----------|------|
| **서버 상태** (API 응답, CRUD) | React Query |
| **클라이언트 상태** (UI, 로컬) | Zustand |

- 서버 상태를 Zustand에 캐싱 **금지** ❌
- DevTools는 **반드시** `getDevtoolsConfig()`로 조건부 활성화 (개발 환경에서만)

```typescript
// ✅ GOOD — DevTools 조건부 활성화
create(devtools((set) => ({ ... }), getDevtoolsConfig("storeName")));

// ❌ FORBIDDEN — 항상 활성화
create(devtools((set) => ({ ... }), { name: "storeName" }));
```

---

## 환경 변수 규칙

`import.meta.env.VITE_*` 문자열을 컴포넌트·훅에서 직접 참조하지 않는다. `ENV` 상수 레지스트리로 중앙 관리한다.

```typescript
// src/constants/env.ts

// ✅ GOOD — 상수 레지스트리
export const ENV = {
  API_URL: import.meta.env.VITE_API_URL as string,
  WS_URL: import.meta.env.VITE_WS_URL as string,
} as const;

export const IS_PRODUCTION = import.meta.env.MODE === 'production';

// ✅ 사용
import { ENV } from '@/constants/env';
axiosInstance.defaults.baseURL = ENV.API_URL;

// ❌ FORBIDDEN — 문자열 직접 접근
import.meta.env.VITE_API_URL  // 오타 검증 없음, 중복 위험
```

**규칙:**
- 모든 `import.meta.env.*` 접근은 `src/constants/env.ts`에서만 정의
- `IS_PRODUCTION` 상수로 환경 분기 통일 — `import.meta.env.MODE === 'production'` 중복 금지
- 미정의 변수는 `as string` 캐스팅 시 `undefined`가 될 수 있으므로 필요한 경우 런타임 검증 추가

---

## 금지 패턴 요약

| 패턴 | 대안 |
|------|------|
| 컴포넌트에서 `axiosInstance` 직접 호출 | API 함수 + hook 사용 |
| `[key: string]: any` index signature | 명확한 필드 정의 |
| `as any` (사유 없이) | 올바른 타입 정의 |
| API 함수 제네릭 생략 | `axiosInstance.get<T>(url)` |
| `.js` / `.jsx` 신규 파일 | `.ts` / `.tsx` 사용 |
| useQuery `onSuccess` / `onError` 옵션 | `useEffect` 또는 컴포넌트 옵션 |
| `isSaving` 수동 boolean | `mutation.isPending` |
| mutation hook에서 toast/alert | 소비 컴포넌트에서 처리 |
| 이중 invalidateQueries | hook 또는 컴포넌트 한 곳에서만 |
| 서버 상태를 Zustand에 저장 | React Query로 관리 |
| `enum` / `const enum` 사용 | `as const` 객체 + 타입 추출 패턴 |
| `import.meta.env.VITE_*` 직접 참조 | `ENV` 상수 레지스트리 (`src/constants/env.ts`) |
| `import.meta.env.MODE === 'production'` 중복 | `IS_PRODUCTION` 상수 |

---

## Enum 규칙

`enum` / `const enum` 사용 금지. `as const` 객체 + 타입 추출 패턴을 사용합니다.

```typescript
// ✅ GOOD
export const OrderStatus = {
  PENDING: 'PENDING',
  COMPLETED: 'COMPLETED',
  CANCELLED: 'CANCELLED',
} as const;

export type OrderStatus = (typeof OrderStatus)[keyof typeof OrderStatus];

// ❌ FORBIDDEN
enum OrderStatus { PENDING, COMPLETED, CANCELLED }
const enum OrderStatus { PENDING = 'PENDING' }
```

**이유:**
- `enum`은 IIFE로 컴파일 → webpack/vite/esbuild의 트리쉐이킹 대상에서 제외될 수 있음 (클라이언트 번들에서 실질적 영향)
- `const enum`은 `isolatedModules: true`(Vite 기본값) 환경에서 동작 불가
- `as const`는 순수 객체 리터럴 → 번들러가 사이드 이펙트 없음을 확신하고 제거 가능

---

## 스킬

- `/add-api-domain {도메인명}` — API 레이어 4파일 보일러플레이트 자동 생성
- `/ts-type-check [파일경로]` — TypeScript 타입 규칙 위반 검사

---

## (팀 모노레포 룰에서 통합) 추가 FE 규칙
<!-- from iCloud/claude/docs/team-java-rules.md § Frontend Rules — fe-rules.md에 없던 항목만 -->

### Boolean 비교 — strict equality

```js
// ✅ GOOD
area?.use_xxx_yn === true

// ❌ BAD (FORBIDDEN) — BE는 boolean → true/false 직렬화
area?.use_xxx_yn === "1"
area?.use_xxx_yn === 1
```

### useEffect deps — 객체 전체 포함 금지

```js
// ❌ BAD — 참조 변경마다 재실행 → 무한 루프 위험
useEffect(() => { ... }, [fetchedCustomer, customer]);

// ✅ GOOD — 원시값(id)만 사용
// eslint-disable-next-line react-hooks/exhaustive-deps
useEffect(() => { ... }, [fetchedCustomer?.id, customer?.id]);
```

### API 응답 필드 타입 가드

API 응답 필드는 `number | string` 가능 → `.trim()` 직접 호출 금지.

```js
// ❌ BAD
field.trim()

// ✅ GOOD
String(field ?? "").trim()
```
