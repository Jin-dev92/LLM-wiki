---
type: project
title: fanddle-server
summary: Fanddle 플랫폼 백엔드. Node/TS + Express + MySQL/Sequelize, yarn workspaces 모노레포(~30 패키지). back/admin/partners 3개 서버, AWS Elastic Beanstalk 배포.
created: 2026-06-18
updated: 2026-06-18
visibility: private
provenance: extracted
status: active
tags: [project, nodejs, express, sequelize, mysql, monorepo, backend]
source_file: fanddle-server (README.md, package.json, packages/)
---

## 목표
Fanddle 서비스의 프로덕션 백엔드. 커머스/결제/포인트·쿠폰·환율·인앱결제 등 도메인을
다루는 다중 서버(app/admin/partners) 시스템.

## 스택 / 아키텍처
- **런타임**: Node.js 18/20, TypeScript, **Express**
- **DB**: MySQL + **Sequelize** (sequelize-typescript) — `@packages/database`에 모델 ~176개
- **모노레포**: yarn workspaces (`packages/*`), 약 30개 패키지
- **큐/캐시**: Redis + BullMQ (`@packages/redis`)
- **배포**: GitHub Actions → AWS Elastic Beanstalk (nginx via `.platform`), 이미지 리사이징 Lambda(`resize/`)
- **배포 브랜치**: `deploy-dev-integration-v3`(개발), `deploy-prod-v3`(실서버)
- **테스트**: Jest

## 서버(앱) 3종
- `@packages/back` — 앱(클라이언트) 서버 (express-rate-limit 적용)
- `@packages/admin` — 관리자 서버
- `@packages/partners` — 파트너스 서버

## 주요 패키지 그룹
- **도메인**: coin(포인트/토리), coupon, currency(환율), payment, portone·portone-v1(PG), iap(인앱결제), delivery-tracker(배송추적), tag
- **인프라**: database, redis(+BullMQ), logger, http-result, environment, i18n, types, utils, api(axios)
- **연동**: auth(AWS SES/SNS), media(AWS S3), fcm, google-api, openai, sns(소셜로그인 검증), slack, translate(AWS)
- **레거시**: node-phpass(PHP 시절 비밀번호 호환)

## 비고
- 현재 위키 `rules/`에는 이 스택(Express/Sequelize)에 직접 맞는 룰이 없다.
  필요 시 Express/Sequelize 컨벤션을 별도 `rules/stacks/`로 추출하는 작업이 후보.
- 비교: estate-server(NestJS+Prisma)와 결이 다름 — fanddle은 Express+Sequelize 모노레포.
  (estate-server 컨텍스트 PR 머지 후 estate-server 노트로 위키링크 상호 연결 권장)
