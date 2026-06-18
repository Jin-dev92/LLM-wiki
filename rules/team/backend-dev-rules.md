---
type: rule
title: 팀 개발 규칙 (Git/PR + Java/Spring 백엔드)
summary: 팀 전체 공유 규칙. Git 브랜치 보호·커밋/PR 컨벤션 + Java/Spring 표준(QueryDSL, DTO record, DDD 패키지, EntityChangeLog).
created: 2026-06-18
updated: 2026-06-18
visibility: team
scope: team
applies_to: [java, spring, querydsl, git]
tags: [backend, java, spring, team, rules]
source_file: iCloud/claude/docs/team-java-rules.md
---

# Team Development Rules

> 팀 전체가 공유하는 개발 규칙. Claude Code가 참조합니다.

---

## Git Rules

### Branch Protection

| Branch Type | Push Allowed |
|----------|--------------|
| `main` | **FORBIDDEN** |
| `dev` | **FORBIDDEN** |
| `release`, `stg` | **FORBIDDEN** |
| `feature/*`, `fix/*`, `hotfix/*` | PR only |
| `conf/*`, `refactor/*`, `chore/*`, `docs/*` | PR only |

**Before push**: `git branch --show-current`

### Commit Message

**Format**: `type: 내용`

| Type | Description |
|------|-------------|
| `feature` | 새로운 기능 |
| `fix` | 버그 수정 |
| `refactor` | 리팩토링 |
| `docs` | 문서 작업 |
| `conf` | 설정 파일 변경 |
| `test` | 테스트 코드 |
| `style` | 코드 스타일 (로직 변경 없음) |
| `chore` | 빌드, 기타 작업 |
| `hotfix` | 긴급 장애 대응 |

**Examples**:
```
feature: 사용자 인증 기능 추가
fix: NPE 오류 수정
refactor: UserService 로직 개선
```

### PR Rules

**Title Format**: `[THUB-XXX] type: description`

- Base branch: always `dev` (except hotfix)
- Single task per PR (기능 혼합 금지)
- Squash commits before push (불필요한 커밋 정리)
- Rebase from dev before PR

**Checklist**:
- [ ] 테스트 코드 작성 완료
- [ ] 로컬 빌드 통과
- [ ] dev 브랜치 rebase 완료
- [ ] Spotless 검증 통과

---

## Java Standards

### QueryDSL — `.where()` comma 금지

PR 리뷰에서 가장 많이 지적되는 항목.

```java
// ✅ GOOD
.where(entity.branchName.eq(branchName)
    .and(entity.deletedAt.isNull()))

// ❌ BAD (FORBIDDEN)
.where(entity.branchName.eq(branchName), entity.deletedAt.isNull())
```

### QueryDSL — JPQL `@Query` 직접 작성 금지

```java
// ❌ FORBIDDEN
@Query("SELECT x FROM Xxx x WHERE x.name = :name")
List<Xxx> findByName(@Param("name") String name);
```

### QueryDSL — `JPAQueryFactory` 사용 규칙

`JPAQueryFactory` 직접 사용을 **지양**한다. 조회 유형에 따라 아래 우선순위를 따른다.

| 우선순위 | 조회 유형 | 방법 | `JPAQueryFactory` |
|---------|----------|------|-------------------|
| 1순위 | 단일 엔티티 단순 조건 | Spring Data JPA **메서드 쿼리** | 사용 안 함 |
| 2순위 | 단일 엔티티 동적 조건 | QueryDSL **Custom Repository** | `*RepositoryImpl`에서만 허용 |
| 3순위 | **다중 엔티티 복합 조회** (JOIN, 통계, 리포트) | QueryDSL **Custom Repository** | `*RepositoryImpl`에서만 허용 |

#### Service 계층 — `JPAQueryFactory` 금지

```java
// ❌ FORBIDDEN — Service에서 JPAQueryFactory 직접 사용
@Service
@RequiredArgsConstructor
public class UserService {
  private final JPAQueryFactory queryFactory; // ❌ 금지
}
```

#### Repository 계층 — Spring Data JPA 메서드 쿼리 우선 + Custom Repository

단순 조건은 메서드 쿼리, 집계/동적/다중 엔티티 조회는 Custom Repository로 분리한다.
아래는 `ClinicRoom` (처치실) 도메인의 실제 프로젝트 예시.

```java
// ✅ 1. Custom Repository 인터페이스 — 커스텀 메서드 선언
public interface ClinicRoomRepositoryCustom {
  /** 지점별 처치실 최대 정렬순서 조회 (삭제되지 않은 항목 대상) */
  Integer findMaxShowOrderByBranchName(String branchName);
}

// ✅ 2. Custom Repository 구현체 — JPAQueryFactory 허용
@Repository
@RequiredArgsConstructor
public class ClinicRoomRepositoryImpl implements ClinicRoomRepositoryCustom {
  private final JPAQueryFactory queryFactory;

  @Override
  public Integer findMaxShowOrderByBranchName(String branchName) {
    Integer result = queryFactory
        .select(clinicRoom.showOrder.max())
        .from(clinicRoom)
        .where(clinicRoom.branchName.eq(branchName)
            .and(clinicRoom.deletedAt.isNull()))
        .fetchOne();
    return result != null ? result : 0;
  }
}

// ✅ 3. 메인 Repository — JpaRepository + Custom 인터페이스 반드시 상속
public interface ClinicRoomRepository
    extends JpaRepository<ClinicRoom, Long>, ClinicRoomRepositoryCustom {

  // 단순 조건은 메서드 쿼리
  List<ClinicRoom> findByBranchNameAndDeletedAtIsNullOrderByShowOrder(String branchName);
  List<ClinicRoom> findByBranchNameAndUseYnAndDeletedAtIsNullOrderByShowOrder(
      String branchName, boolean useYn);
  boolean existsByBranchNameAndContentsAndDeletedAtIsNull(
      String branchName, String contents);
}
```

**규칙 요약:**
- **Service/Controller**: `JPAQueryFactory` 사용 금지
- **Repository**: Spring Data JPA 메서드 쿼리 우선, `JPAQueryFactory` 지양
- **다중 복합 객체 조회**: QueryDSL Custom Repository(`*RepositoryImpl`)에서만 `JPAQueryFactory` 허용
- **마이그레이션**: 신규 코드는 필수 준수, 기존 코드는 해당 파일 수정 시 함께 리팩토링

### DTO Rules

- **Record type 사용** (Entity 제외)
- **Inner Class 금지** → 별도 파일로 분리 (`models/dto/`)

| 조건 | 타입 |
|------|------|
| 필드 ≤ 15개 | `record` |
| 필드 16개 이상 | `class + @Builder + @Getter` |
| Spotless 포맷 깨지는 경우 | `class + @Builder + @Getter` |

```java
// ✅ 소형 DTO — record
public record FeatureRoleEntry(String feature, String role) {}

// ✅ 대형 DTO (16+) — class+Builder
@Getter @Builder
public class AreaDto { /* 30+ fields */ }

// ❌ FORBIDDEN — Inner Class
public class UserController {
    public static class UserResponse { ... }
}
```

### Package Structure (DDD)

```
domain/{domain}/
├── controller/
├── service/
├── repository/
├── models/
│   ├── dto/
│   ├── enums/
│   └── ...
└── exception/
```

### EntityChangeLog — Entity 변경 시 필수

Entity 관련 **모든** 변경(생성·수정·삭제·복원·상태 전이) 시 이벤트 발행 필수.

```java
// ✅ GOOD
Xxx saved = xxxRepository.save(entity);
entityChangeLogService.publish(EntityType.XXX, ActionType.CREATE, saved.getId());

// ❌ BAD — 이벤트 발행 없이 저장만
xxxRepository.save(entity);
```

- `EntityType` / `ActionType` 없으면 추가 필요
- 테스트도 함께 추가/갱신
- 가이드: `docs/backend/audit/entity-change-log-guide.md`

### @ManyToOne null guard 필수

`@ManyToOne(fetch = LAZY)` 필드는 항상 null 가능.

```java
// ✅ GOOD — 상황에 맞게 선택
.filter(info -> info.getOptionCode() != null)                      // Stream
entity.getX() == null ? null : entity.getX().getField()            // 삼항
if (info.getOptionCode() != null) { ... }                          // If guard

// ❌ BAD — null guard 없이 체이닝 (NPE 위험)
entity.getOptionCode().getOptionCode()
```

### ObjectMapper.convertValue() 주의사항

글로벌 `PropertyNamingStrategy`를 따름 — 클래스의 `@JsonNaming` 무시됨.

```java
// ❌ 문제 — 글로벌 SNAKE_CASE가 camelCase 필드명 덮어씀
AreaDto dto = objectMapper.convertValue(area, AreaDto.class);

// ✅ 해결 — 새 ObjectMapper (글로벌 설정 없음)
ObjectMapper plain = new ObjectMapper();
AreaDto dto = plain.convertValue(area, AreaDto.class);
```

### Swagger @Operation — 변경된 Controller 엔드포인트 필수

수정한 도메인의 Controller 메서드에 `@Operation` 없으면 추가.

```java
// ✅ GOOD
@Operation(summary = "예약 알림 설정 조회")
@GetMapping("/reservation-alert-config")
public ResponseEntity<XxxResponse> getConfig(...) { ... }

// ❌ BAD — @Operation 없음
@GetMapping("/reservation-alert-config")
public ResponseEntity<XxxResponse> getConfig(...) { ... }
```

- 신규 엔드포인트: 반드시 추가
- 기존 엔드포인트 수정 시: 없으면 이번 PR에서 함께 추가
- `summary` 한 줄로 기능 요약 (한국어 OK)

### JavaDoc

- Plain text only (HTML 태그 금지)
- 1-2 lines max
- 명확한 경우 생략 가능

```java
// GOOD
/** 사용자 ID로 조회 */
public User findById(Long id) { ... }

// BAD - too verbose
/**
 * <p>사용자를 ID로 조회합니다.</p>
 * @param id 사용자 ID
 * @return User 객체
 */
```

---

## JSON Serialization Rules

### Jackson 기본 설정

JacksonConfig에서 **SNAKE_CASE**가 기본값. DTO에 `@JsonNaming` 미지정 시 `snake_case`로 직렬화됨.

### @JsonNaming 적용 규칙

| 상황 | 규칙 | 이유 |
|------|------|------|
| **Legacy** (기존 FE-BE 정의된 DTO) | **변경 금지** | 일괄 변경 시 리스크 큼 |
| **FE-BE 동시 수정 기회** | **camelCase로 리팩토링** 권장 | 동일 도메인 DTO 변경 시 기회 활용 |
| **신규 feature / 신규 도메인** | **처음부터 camelCase** | 표준 준수 |

### 구현 방법

```java
// 신규 DTO 또는 리팩토링 대상
@JsonNaming(PropertyNamingStrategies.LowerCamelCaseStrategy.class)
public record UserResponse(Long userId, String userName) {}

// Legacy - 기존 snake_case 유지 (변경 X)
public record OldApiResponse(Long user_id, String user_name) {}
```

### 주의사항

- **일괄 @JsonNaming 변경 금지** → 범위가 방대하여 FE 호환성 문제 발생
- 리팩토링 시 **반드시 FE와 동시 수정** 필요
- 신규 작업 시 **camelCase 표준 준수**

---

## Exception Rules

### 커스텀 Exception 작성

```java
public class MyNotFoundException extends RuntimeException {
  public MyNotFoundException(String message) {
    super(message);
  }
}
```

### Exception 분류 및 HTTP Status

| 타입 | HTTP | 로그 레벨 | 예시 |
|------|------|---------|------|
| 비즈니스 예외 | 400 | WARN | NotFoundException, ValidationException |
| Rate Limit | 429 | WARN | RateLimitExceededException |
| 동시성 충돌 | 409 | WARN | LockTimeoutException |
| 시스템 에러 | 500 | ERROR | NullPointerException, SQLException |

---

## Logging Rules

### Logger 선언

```java
@Slf4j  // Lombok 사용
@Service
public class MyService { ... }
```

### 로그 레벨 사용

| 레벨 | 용도 | Sentry 전송 |
|------|------|------------|
| ERROR | 시스템 에러, 예상치 못한 예외 | O |
| WARN | 비즈니스 예외, 예상된 오류 | X |
| INFO | 중요 작업 완료 | X |
| DEBUG | 상세 추적, 개발 디버깅 | X |

### 로깅 패턴

```java
// ERROR - 예외 객체 마지막에 전달 (스택 트레이스 포함)
log.error("Unexpected error - uri: {}, error: {}", uri, ex.getMessage(), ex);

// WARN - 메시지만
log.warn("Business exception: {}", ex.getMessage());

// INFO - 작업 결과
log.info("User created: userId={}, name={}", id, name);
```

---

## Forbidden Patterns

| Pattern | Reason | Alternative |
|---------|--------|-------------|
| `System.out.println` | 로그 추적 불가 | `log.info/debug` |
| `@Autowired` field | 테스트 어려움 | Constructor injection |
| Empty catch `catch(e){}` | 에러 무시 | 로깅 또는 재throw |
| `@ts-ignore`, `as any` | 타입 안전성 파괴 | 올바른 타입 정의 |
| Raw SQL in Controller | 계층 분리 위반 | Repository/Service 사용 |

---

## Code Style

### Spotless (강제)

- **Formatter**: Google Java Format (AOSP style)
- **Indent**: 2 spaces
- **Import order**: `java` → `javax` → `org` → `lombok` → `com`

### Verification

```bash
just spotless-check   # 검증
just spotless-fix     # 자동 수정
```

---

## Document Location

| Location | Allowed |
|----------|---------|
| `{project}/docs/` | **O** |
| `{submodule}/docs/` | **X** (FORBIDDEN) |

**Subfolders**: `backend/`, `database/`, `frontend/`, `plans/`, `test/`, `qa/`, `infra/`, `inspect/`, `guides/`

---

## TDD Requirements

- Java 파일 변경 시 UTIL, SERVICE 성 파일 및 기능에 대해서는 반드시 **테스트 코드 필수**
- 테스트 미작성 시 PR 리뷰 실패

---

## Database Rules

### Liquibase (MANDATORY)

**DB 조회 제외한 모든 DDL/마이그레이션은 Liquibase로 수행**

| 작업 | 방법 |
|------|------|
| 테이블 생성/수정/삭제 | Liquibase |
| 컬럼 추가/수정/삭제 | Liquibase |
| 인덱스 생성/삭제 | Liquibase |
| 데이터 조회 | SQL (MCP 허용) |

### Changelog 작성 규칙

| 항목 | 규칙 | 예시 |
|------|------|------|
| 파일명 | `YYYYMMDD-NNN-description.yaml` | `20241127-001-add-user-table.yaml` |
| changeset ID | `YYYYMMDD-NNN` | `20241127-001` |
| author | GitHub username | `ttc-cbj` |

```yaml
databaseChangeLog:
  - changeSet:
      id: 20241127-001
      author: ttc-cbj
      changes:
        - createTable:
            tableName: ttc_example
```

### 테이블/컬럼 네이밍

| 항목 | 규칙 | 예시 |
|------|------|------|
| 테이블명 | `snake_case`, prefix: `ttc_` | `ttc_customer_grade` |
| 컬럼명 | `snake_case` | `created_at`, `branch_name` |
| PK | `id` (BIGINT AUTO_INCREMENT) | - |
| FK | `{table}_id` | `customer_id` |

---

## API/Service Design Rules

### REST API 규칙

| Method | 용도 | URL 패턴 |
|--------|------|----------|
| GET | 목록 조회 | `/api/{resources}` |
| GET | 단건 조회 | `/api/{resources}/{id}` |
| POST | 생성 | `/api/{resources}` |
| PUT | 전체 수정 | `/api/{resources}/{id}` |
| PATCH | 부분 수정 | `/api/{resources}/{id}` |
| DELETE | 삭제 | `/api/{resources}/{id}` |

### Service 메서드 네이밍

| 동작 | 패턴 | 예시 |
|------|------|------|
| 조회 | `find*` | `findById`, `findByBranchName` |
| 생성 | `create*` | `createUser` |
| 수정 | `update*` | `updateUser` |
| 삭제 | `delete*` | `deleteById` |
| 저장 (JPA) | `save` | `repository.save(entity)` |

**FORBIDDEN**: `get*` for DB queries → use `find*`

```java
// GOOD
public User findById(Long id) { ... }
public List<User> findByBranchName(String branchName) { ... }

// BAD
public User getById(Long id) { ... }  // ❌ get 대신 find 사용
```

---

## NestJS Rules

> **상세 규칙**: [`.claude/docs/nestjs-rules.md`](.claude/docs/nestjs-rules.md) 참조
> Tech Stack: NestJS, TypeORM, JWT/Session, class-validator, Swagger

### 핵심 원칙

| 항목 | 규칙 |
|------|------|
| 모듈 구조 | Feature Module per domain (DDD) |
| DTO 검증 | `class-validator` 데코레이터 필수 |
| Swagger | `@ApiOperation` + `@ApiProperty` 필수 |
| 환경변수 | `ConfigService.get()` 경유 (`process.env` 직접 접근 금지) |
| Soft Delete | `softDelete()` 사용 (`delete()` 금지) |
| 트랜잭션 | 다중 save는 `dataSource.transaction()` 필수 |
| 테스트 | Service 변경 시 `.service.spec.ts` 필수 |

---

## Frontend Rules

> **상세 규칙**: [`.claude/docs/fe-rules.md`](.claude/docs/fe-rules.md) 참조

### State Management

| State Type | Tool | Example |
|------------|------|---------|
| **Server State** | React Query | API data, CRUD |
| **Client State** | Zustand | UI state, auth |

**FORBIDDEN**: Redux for data fetching, `useEffect + useState` for API calls

### Directory Structure

```
src/
├── api/{domain}.ts              # API functions (JSDoc: HTTP Method + Endpoint)
├── types/api/{domain}.types.ts  # Type definitions
├── hooks/queries/use{Domain}Queries.ts
└── hooks/mutations/use{Domain}Mutations.ts
```

### API Function Naming

| 동작 | 패턴 | 예시 |
|------|------|------|
| 목록 조회 | `get{Resource}s` | `getAreas` |
| 단건 조회 | `get{Resource}ById` | `getBannerById` |
| 생성/수정/삭제 | `create/update/delete{Resource}` | `createArea` |

### Boolean 비교 — strict equality

```js
// ✅ GOOD
area?.use_xxx_yn === true

// ❌ BAD (FORBIDDEN) — BE는 boolean → true/false 직렬화
area?.use_xxx_yn === "1"
area?.use_xxx_yn === 1
```

### Mutation Hook — UI 피드백 직접 호출 금지

```js
// ✅ GOOD — hook은 캐시 무효화만
onSuccess: () => {
  queryClient.invalidateQueries(QueryKeys.list(branchName));
}

// ❌ BAD — hook에서 직접 UI 호출 (FORBIDDEN)
onSuccess: () => {
  queryClient.invalidateQueries(...);
  enqueueSnackbar("완료"); // 소비 컴포넌트의 onSettled에서 처리할 것
}
```

### 이중 캐시 무효화 금지 (Race Condition)

mutation `onSuccess`에서 처리하면 소비 컴포넌트 `useEffect`에서 중복 호출 금지.
순서: stale → 최신 → **stale 재호출** → 오래된 데이터 표시.

```js
// ❌ BAD — mutation + useEffect 양쪽에서 invalidate
onSuccess: () => queryClient.invalidateQueries(QueryKeys.list(branchName))

// 소비 컴포넌트 — 중복 금지
useEffect(() => {
  queryClient.invalidateQueries(...); // FORBIDDEN
}, [fetchedData]);
```

예외: `fetchQuery` → `setQueryData` 패턴은 OK (네트워크 호출 없이 캐시 직접 업데이트)

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

### Query/Mutation Hooks

```typescript
// Query Key 패턴
export const userQueryKeys = {
  all: ['users'] as const,
  list: (branchName: string) => ['/api/users', { branchName }] as const,
};

// Query Hook
export const useGetUsers = (branchName: string | null) =>
  useQuery({
    queryKey: ['/api/users', { branchName }],
    queryFn: () => getUsers(branchName),
    enabled: !!branchName
  });

// Mutation Hook (반드시 invalidate)
export const useCreateUser = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: createUser,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: userQueryKeys.all })
  });
};
```

### Field Naming (신규 도메인)

**camelCase ONLY** (FE + BE 통일)

```typescript
// CORRECT
interface CreateUserRequest { userName: string; workStartDate: string; }

// FORBIDDEN
interface Bad { user_name: string; }  // ❌ snake_case
```

### Type Safety

**FORBIDDEN:**
- `[key: string]: any` index signature
- `any`, `unknown` types
- All fields optional when required

### FE FORBIDDEN Patterns

| Pattern | Alternative |
|---------|-------------|
| Redux for server data fetching | React Query |
| Direct `axiosInstance` in components | API function + hook |
| Query keys without `/api/` endpoint | `['/api/endpoint', params]` |
| Types without Request/Response suffix | `CreateUserRequest`, `UserResponse` |
| Mutation without cache invalidation | `onSuccess: invalidateQueries` |

---

## justfile Commands

| Command | Description |
|---------|-------------|
| `just spotless-check` | 코드 스타일 검증 |
| `just spotless-fix` | 코드 스타일 자동 수정 |
| `just install-hooks` | Git Hooks 설치 |
| `just start-local` | React 개발서버 (local) |
| `just start-dev` | React 개발서버 (dev) |
| `just base-status [env]` | Liquibase 상태 확인 |
| `just base-update [env]` | Liquibase 마이그레이션 |
