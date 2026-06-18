---
type: rule
title: Prisma DB 규칙 (PostgreSQL)
summary: Prisma+PostgreSQL DB 규칙 — cuid id, createdAt/updatedAt, 논리삭제(deletedAt + @@index), enum in schema, prisma migrate(타임스탬프_설명) 마이그레이션, DATABASE_URL env.
created: 2026-06-18
updated: 2026-06-18
visibility: team
scope: stack
applies_to: [prisma, postgresql, nestjs]
tags: [backend, db, prisma, rules]
source_file: estate-server (prisma/schema.prisma, prisma/migrations, soft-delete 설계)
---

## Prisma DB 규칙 (PostgreSQL)

> NestJS+Prisma 프로젝트의 DB 규칙. (Java/Spring의 Liquibase 규칙을 대체하는 Prisma 버전)
> 관련: [[rules/stacks/nestjs]]

### 모델 컨벤션

| 항목 | 규칙 | 예시 |
|------|------|------|
| PK | `String @id @default(cuid())` | `id String @id @default(cuid())` |
| 타임스탬프 | `createdAt DateTime @default(now())`, `updatedAt DateTime @updatedAt` | |
| 필드명 | `camelCase` | `ownerId`, `passwordHash` |
| enum | schema 내 `enum` 선언 후 필드에 사용 | `role Role @default(TENANT)` |
| 관계 | `@relation(fields/references)` 명시 | |

### 논리삭제 (Soft Delete)

물리삭제(`delete`) 대신 **`deletedAt DateTime?` 컬럼**으로 논리삭제한다.

```prisma
model User {
  id        String    @id @default(cuid())
  // ...
  deletedAt DateTime?

  @@index([deletedAt])   // 활성 행 조회 성능
}
```

- 조회 시 기본적으로 `where: { deletedAt: null }` 로 활성 행만 노출.
- 삭제는 `update({ data: { deletedAt: new Date() } })`.
- 논리삭제 대상 엔티티에는 `@@index([deletedAt])` 부여.

### 마이그레이션

| 항목 | 규칙 |
|------|------|
| 도구 | **Prisma Migrate** (`prisma migrate dev` / `deploy`). 수기 DDL 금지 |
| 디렉터리명 | `YYYYMMDDHHMMSS_설명` (Prisma 생성) — 예: `20260613085931_add_soft_delete` |
| 단위 | 마이그레이션 1개 = 논리적 변경 1개 (기능/스펙 단위) |
| lock | `migration_lock.toml` 추적 |

### 접속/환경

- 접속 URL은 `env("DATABASE_URL")` 로만. 하드코딩 금지([[rules/stacks/nestjs]] 설정키 상수화).
- `generator client { provider = "prisma-client-js" }`, `datasource db { provider = "postgresql" }`.

---
