---
type: rule
title: NestJS 일반 규칙
summary: NestJS 핵심 원칙 — Feature Module(DDD), class-validator DTO 검증, Swagger 필수, ConfigService 경유, softDelete, 트랜잭션. (테스트는 nestjs-test 참조)
created: 2026-06-18
updated: 2026-06-18
visibility: team
scope: stack
applies_to: [nestjs, typeorm, typescript]
tags: [backend, nestjs, rules]
source_file: iCloud/claude/docs/team-java-rules.md (NestJS 섹션)
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
