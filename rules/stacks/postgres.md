---
type: rule
title: PostgreSQL 규칙 (MySQL 이관 관점)
summary: MySQL에서 넘어올 때 실제로 깨지는 지점 중심의 PostgreSQL 규칙 — 식별자 대소문자, 타입 매핑(tinyint/datetime/text), AUTO_INCREMENT 대체, ON UPDATE 부재, upsert(ON CONFLICT), 인덱스·격리수준 차이. ORM 무관한 DB 레이어 규칙.
created: 2026-08-19
updated: 2026-08-19
visibility: team
scope: stack
applies_to: [postgresql, mysql, migration]
tags: [backend, db, postgres, mysql, rules]
source_file: 신규 작성 (MySQL→PostgreSQL 마이그레이션 대비)
---

## PostgreSQL 규칙 (MySQL 이관 관점)

> ORM에 종속되지 않는 **DB 레이어 규칙**이다. Prisma 사용 시의 모델 컨벤션은
> [[rules/stacks/prisma]], 애플리케이션 규칙은 [[rules/stacks/nestjs]] 참조.
> MySQL 스키마를 옮길 때 조용히 깨지는 지점을 우선 다룬다.

### 식별자 대소문자 — 가장 흔한 사고

PostgreSQL은 따옴표 없는 식별자를 **소문자로 접는다**. MySQL의 `camelCase` 테이블/컬럼명을
그대로 옮기면 `"userId"`처럼 **항상 따옴표를 붙여야만** 접근되는 컬럼이 된다.

- 신규 스키마는 **`snake_case`로 통일**한다. 애플리케이션의 camelCase는 ORM 매핑으로 해결한다.
- 기존 camelCase를 유지해야 하면 그 사실을 스키마 주석에 남기고 **전 쿼리에서 따옴표를 강제**한다. 절반만 따옴표를 쓰면 런타임에서만 터진다.

### 타입 매핑 (기계적으로 옮기면 안 되는 것들)

| MySQL | PostgreSQL | 주의 |
|---|---|---|
| `tinyint(1)` | `boolean` | MySQL은 0/1 정수다. 그대로 `smallint`로 옮기면 애플리케이션의 truthy 판정이 어긋난다 |
| `datetime` | `timestamptz` | `timestamp`(without tz)를 고르지 않는다. MySQL `datetime`은 TZ 정보가 없어 **이관 시 어느 TZ로 해석할지 명시**해야 한다 |
| `int unsigned` | `bigint` | PostgreSQL에 unsigned가 없다. 범위 초과를 피하려면 한 단계 올린다 |
| `varchar(n)` / `text` | `text` | PostgreSQL은 `text`와 `varchar(n)`의 성능 차가 없다. 길이 제한은 검증 계층에서 건다 |
| `enum('a','b')` | `enum` 타입 또는 `text` + CHECK | PostgreSQL enum은 값 추가는 쉽지만 **삭제·순서 변경이 어렵다**. 자주 바뀌면 `text` + CHECK |
| `json` | `jsonb` | `json`은 원문 보존만 한다. 조회·인덱싱이 필요하면 `jsonb` |
| `double` | `double precision` / `numeric` | 금액은 부동소수 금지. `numeric` |

### AUTO_INCREMENT 대체

- 신규 스키마 PK는 `bigint GENERATED ALWAYS AS IDENTITY` (SQL 표준).
- `serial`은 레거시 표기다. 신규에는 쓰지 않는다.
- **데이터 이관 후 시퀀스 현재값을 반드시 맞춘다.** 빠뜨리면 첫 쓰기에서 PK 충돌이 난다.
  `setval(pg_get_serial_sequence('users','id'), max(id))` 형태로 테이블마다 실행하고,
  이관 체크리스트의 필수 항목으로 둔다.

### `ON UPDATE CURRENT_TIMESTAMP` 가 없다

MySQL의 자동 갱신 컬럼이 PostgreSQL에는 없다. 셋 중 하나를 **명시적으로** 고른다.

1. ORM에 맡긴다 (Prisma `@updatedAt`) — 애플리케이션을 우회하는 쓰기에는 적용되지 않는다
2. `BEFORE UPDATE` 트리거 — DB 레벨 보장이 필요할 때
3. 애플리케이션에서 매번 세팅

> 이관 후 "updated_at이 안 변한다"는 버그의 대부분이 이 차이다.

### upsert

MySQL의 `INSERT ... ON DUPLICATE KEY UPDATE`에 대응하는 것은 `ON CONFLICT` 절이다.

- **대상 제약을 명시해야 한다** — `ON CONFLICT (email)` 또는 `ON CONFLICT ON CONSTRAINT ...`.
  MySQL처럼 아무 unique 키나 알아서 잡아주지 않는다.
- 갱신값은 `EXCLUDED.<컬럼>`으로 참조한다.
- 충돌 시 무시하려면 `DO NOTHING`.

### 인덱스

- **부분 인덱스**를 쓴다. 논리삭제 테이블의 활성 행 조회는 `WHERE deleted_at IS NULL` 조건부 인덱스가 전체 인덱스보다 작고 빠르다.
- 대소문자 무시 검색은 MySQL의 기본 collation에 기대지 않는다. PostgreSQL 기본 collation은 **대소문자를 구분한다** — `citext` 확장이나 `lower()` 표현식 인덱스를 명시적으로 쓴다.
- 운영 중 인덱스 생성은 `CONCURRENTLY` 옵션으로 테이블 락을 피한다. 단 트랜잭션 안에서는 쓸 수 없다.

### 트랜잭션·락

- 기본 격리수준: MySQL InnoDB는 `REPEATABLE READ`, PostgreSQL은 **`READ COMMITTED`**. MySQL에서 격리수준에 의존하던 로직은 이관 시 재검토 대상이다.
- PostgreSQL은 갱신 충돌 시 **직렬화 실패로 트랜잭션을 되돌린다**. 재시도 경로가 필요하다 → [[retry-idempotency-and-backoff]]
- 긴 트랜잭션은 `VACUUM`을 막아 테이블을 부풀린다. 트랜잭션 안에서 외부 API를 호출하지 않는다.

### 마이그레이션 운영

- DDL이 트랜잭션 안에서 동작한다(MySQL과 다른 점). 마이그레이션이 실패하면 **전체 롤백**되므로 부분 적용 걱정이 없다.
- 단 `CREATE INDEX CONCURRENTLY`·enum 값 추가는 트랜잭션 밖에서만 가능하다 — 별도 마이그레이션 파일로 분리한다.
- 기존 행이 많은 테이블의 **타입 변경은 전체 재작성**이다. 무중단이 필요하면 신규 컬럼 추가 → 백필 → 스위치 순으로 쪼갠다.

## 배경/이유
- 레거시(Express + MySQL) → NestJS + PostgreSQL 이관 대비. 스키마를 기계적으로 옮기면 위 지점들이 **런타임에서만** 드러난다.
- 관련: [[rules/stacks/prisma]] · [[rules/stacks/nestjs]] · [[strangler-fig-migration]]
