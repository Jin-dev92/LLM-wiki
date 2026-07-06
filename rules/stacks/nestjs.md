---
type: rule
title: NestJS 일반 규칙
summary: NestJS 핵심 원칙 — Feature Module(DDD), class-validator DTO, Swagger 필수, 설정키 상수화(const enum), ORM 트랜잭션, 논리삭제(deletedAt). DB 세부는 prisma 룰 참조.
created: 2026-06-18
updated: 2026-06-18
visibility: team
scope: stack
applies_to: [nestjs, typescript, prisma]
tags: [backend, nestjs, rules]
source_file: iCloud/claude/docs/team-java-rules.md (NestJS 섹션) + estate-server/CLAUDE.md (최신화)
---

## NestJS Rules

> Tech Stack 기준: NestJS, Prisma(PostgreSQL), JWT/Passport, class-validator, Swagger.
> DB 세부 규칙은 [[rules/stacks/prisma]] 참조. 테스트는 [[rules/stacks/nestjs-test]] 참조.
> Redis 캐시 규칙은 [[rules/stacks/redis]], 외부 의존성 장애 대응은 [[rules/stacks/resilience]] 참조.

### 핵심 원칙

| 항목 | 규칙 |
|------|------|
| 모듈 구조 | Feature Module per domain (DDD). 예: `src/{auth,property,board,chat,...}` |
| DTO 검증 | `class-validator` 데코레이터 필수 |
| Swagger | 모든 엔드포인트에 `@ApiTags`+`@ApiOperation`+`@ApiResponse` 필수. 인증 라우트는 `@ApiBearerAuth`. DTO 필드는 `@ApiProperty`(enum은 `enum`+`enumName` 명명) |
| 환경변수 | `ConfigService.get/getOrThrow` 경유(`process.env` 직접 접근 금지) + **키를 상수(const enum)로 참조**(하드코딩 문자열 금지) |
| 논리삭제 | 물리삭제 대신 `deletedAt` 컬럼 사용. 상세는 [[rules/stacks/prisma]] |
| 트랜잭션 | 다중 쓰기는 ORM 트랜잭션 API로 묶는다 (Prisma `prisma.$transaction(...)`) |
| 테스트 | Service 변경 시 `.service.spec.ts` 필수 |

### 설정 키 상수화 (매직 스트링 금지)

env 키·Redis 키 prefix·토픽명·메타데이터 키 등 반복되는 매직 스트링은 의미 있는
상수/`const enum`으로 추출해 단일 출처로 관리하고 오타를 컴파일 타임에 잡는다.
(Redis 키 설계 자체의 슬롯/TTL 규칙은 [[rules/stacks/redis]] 참조)

```ts
// ✅ GOOD
config.getOrThrow<string>(ConfigKey.JwtSecret)
// ❌ FORBIDDEN
config.getOrThrow<string>('JWT_SECRET')
```

새 env 키 추가 시 `.env.example`과 `ConfigKey`에 함께 등록한다.

### API 문서화 (변경 시 필수)

엔드포인트 추가·변경·삭제 시 README의 API 표와 PR 본문에 (메서드+경로 / 기능 / 인가)를
명시하고, Swagger 데코레이터와 **병행** 갱신한다.

---
