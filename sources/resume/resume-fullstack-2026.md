---
type: source
title: 김의진 이력서 — 풀스택 개발자 (2026)
summary: 5년 차 풀스택(백엔드~인프라) 개발자 김의진의 이력서. PDF 원문과 1:1로 일치하는 마크다운(이력서 수정 diff 기준용).
created: 2026-06-19
updated: 2026-06-19
visibility: private
url:
author: 김의진
ingested_via: pdf
tags: [resume, profile, career, fullstack]
---

> **이 본문은 `김의진_ 풀스택 개발자.pdf`(4쪽)와 1:1로 일치하도록 작성됨.**
> 이력서를 수정하면 이 파일을 고치고 diff로 변경분을 확인한다. 문구·순서·괄호 기간까지 PDF를 따르며,
> 줄바꿈 정리(랩핑 합치기)와 마크다운 헤딩/볼드만 가공했다. 요약·영구노트 등 위키 메타는 본문 아래 별도.

# 김의진

- 010-9545-0709
- jindevst@gmail.com

5년간 스타트업과 프리랜서를 거치며 백엔드부터 인프라까지 직접 챙기는 풀스택 개발자로 일해왔습니다.
- API 응답 시간을 93% 단축하고, PR 파이프라인을 자동화한 경험처럼 느린 건 빠르게, 없는 건 만들고, 반복되는 건 자동화하는 방식으로 일하는 것을 좋아합니다.
- Backend: NestJS, Node.js, TypeScript, Spring Boot, Java, ExpressJS
- Frontend: React, Next.js, JavaScript, HTML/CSS, Tailwind, styled-components, Lua
- Database: PostgreSQL, MySQL, MongoDB, Redis, Aurora DB
- Infra: AWS (ECS, ECR, RDS, CloudFront, Lambda), Docker, CI/CD (GitHub Actions, CodePipeline)
- Test: Jest, JUnit, Mockito
- Tool: Jira, Notion, Slack, Git, intelliJ
- AI: Claude Code, Gemini, GitHub Copilot
- APM: Sentry, AWS CloudWatch

## 경력 4년 4개월

### TTC
2025.02 - 2026.03 (1년 2개월) 정규직 개발

**THUB AI - THUB 솔루션 내 사용되는 AI 서버 개발 및 운영 (NestJS)**
2025.07 - 2026.03
[주요 역할]
- 전자 차트 요약 자동화 및 시술 상품 추천 AI 서버 개발 및 운영
[핵심 성과]
- 일 약 100건 규모의 환자 차트 데이터를 전처리·정제 후 Gemini API에 전달하는 프롬프트 파이프라인 설계
- 차트 요약 자동화로 수동 차트 업무 제거
- 신규 환자 설문 데이터와 시술 이력을 결합한 시술 상품 추천 시스템 개발

**THUB - B2B, B2C 병원 내 고객 CRM, 전자차트 풀스택 개발 (Spring Boot + ReactJS)**
2025.02 - 2026.03
[주요 역할]
- 병원 고객 CRM 및 전자 차트 기능 개발 및 운영
[핵심 성과]
- 누적 고객 5000명 규모 CRM에서 주요 API 병목 분석, 병렬 처리 + Redis 캐싱 도입으로 응답 시간 15초 → 2초 (87% 단축)
- 데이터 정합성 확보를 위한 Event-Driven 기반 EntityChangeLog 시스템 설계 및 도입, 변경 이력 추적 체계 구축
- JUnit + Mockito 기반 테스트 환경을 주도적으로 구축, given/when/then + Factory 패턴으로 팀 컨벤션 정립
- Retool 통계 대시보드 구축으로 경영진 수동 데이터 요청 업무 완화
- Retool을 이용한 반복 쿼리 작업을 셀프서비스로 전환
- Claude Code + MCP(Sentry, Notion) 연동을 통한 워크플로우 구축
- PR 리뷰 품질 향상/이슈트래킹, 이슈 Fix 후 PR 작성 자동화
- 요구사항 분석, 일정 관리, 코드 리뷰 피드백 등 PM · 시니어와의 실무 협업을 통한 소프트 스킬 성장

### 오로라파이브
2023.05 - 2024.07 (1년 3개월) 정규직

**글로벌 e-commerce 리액트 네이티브 앱 Fanddle의 서버 개발 (1년)**
2023.06 - 2024.06
- 주요 역할
- Node.js 백엔드 개발 및 글로벌 인프라 구축
- 핵심 성과
- 글로벌 서비스 특성상 Read 트래픽이 집중되는 구조를 고려해 Command/Query 분리 및 Read Replica 전략 제안 (서버 2인)
- AWS CloudFront + 다중 리전 배포 기반 글로벌 인프라 구축으로 평균 응답 시간 1.2초 → 0.08초 (93% 단축)
- Jest 기반 테스트 코드 도입 및 커버리지 80% 이상 유지
- Slack + AWS CloudWatch 기반 실시간 모니터링 체계 구축으로 장애 대응 속도 개선
- 환율 변동으로 인한 결제 손실 방지를 위한 한국수출입은행 API 기반 환율 자동 갱신 스케쥴러 및 캐싱 구현
- 앱에서 지원하는 언어 추가를 쉽게 하기 위해 테이블 재설계 및 i18n 도입
- 다국어 번역 자동화 스크립트 제작으로 번역 오류율 80% 감소
- DeliveryTracker GraphQL API 연동으로 운송장 번호 기반 실시간 배송 조회 구현

**Fanddle 백 오피스 신규 개발**
2023.05 - 2023.07
[주요 역할]
- React 기반 백오피스 아키텍처 설계 및 외부 프리랜서 2~3명 협업
[핵심 성과]
- 빌드 최적화를 통한 배포 시간 및 앱 실행 시간 70% 단축
- AWS lambda를 통한 로딩 속도 3.2초 → 0.8초 (80% 개선)
- React Query 캐싱 전략 설계 및 성능 분석

### 프리랜서
2022.01 - 2023.04 (1년 4개월) 정규직

**Electric Vehicle 쇼핑몰 및 전기 충전 구독 서비스 1인 서버 개발 및 유지보수 (1년 3개월)**
2022.01 - 2023.04
[주요 역할]
- 요구사항 분석부터 DB 설계, API 개발, 인프라 구성, 배포까지 전 과정 1인 담당
- 클라이언트와 직접 협업하며 요구사항을 수렴하고 기술 결정 주도
[핵심 성과]
- Spring Boot 운영 중 응답 지연과 배포 복잡도 문제를 진단하고, Django + PostgreSQL로 직접 마이그레이션 결정 및 수행
- 서버 지연 시간 80% 감소, 배포 사이클 간소화
- 충전 구독 서비스의 결제/구독 주기 관리 로직 설계
- 갱신 주기, 상태 전이(활성/만료/해지) 처리를 직접 구현
- Redis 기반 JWT 토큰 블랙리스트 도입으로 로그아웃 후 토큰 탈취 시나리오 차단
- AWS 기반 인프라 직접 구성 및 운영, Swagger로 API 문서화해 클라이언트 커뮤니케이션 비용 절감

**삼천리 도시가스 리액트 백 오피스 페이지 (4개월)**
2022.12 - 2023.04
[핵심 성과]
- React·nivo 기반 데이터 시각화로 가독성 개선
- GitHub Actions 기반 CI/CD 구축으로 배포 시간 30% 단축

### 펄어비스
2018.10 - 2019.04 (7개월) 정규직

**검은 사막 게임 UI 개발 (5개월)**
2018.11 - 2019.04
- 주요 역할
- MMORPG 검은 사막(PC·Mobile·XBOX) UI 컴포넌트 개발
- 핵심 성과
- 대만, 유럽 다국어 관련 로컬라이징 경험
- 모니터링 및 게임 로그 분석을 통한 트러블 슈팅 경험
- 전쟁 컨텐츠에 대한 테스트 케이스 작성

## 학력
한국폴리텍 I 대학 서울정수캠퍼스
2012.03 - 2016.02 졸업 유비쿼터스통신과
정보통신 및 컴퓨터 공학에 관련된 내용을 학습했습니다.

## 스킬
React Java JavaScript Python TypeScript JPA Spring Boot Git HTML
MySQL SQL Django CSS AWS React.js GitHub Node.js Nest.js ORM
Docker Kubernetes PostgreSQL

## 수상/자격증/기타

정보처리기사
2024.12 기타
한국산업인력공단에서 주관하는 소프트웨어 개발 관련 자격증

[DB 자격증] SQLD
2020.12 기타
데이터베이스(Oracle 및 MySql) 관련 자격증이며, 한국데이터베이스진흥센터에서 발행하는 자격증입니다.

[교육] 스마트 웰빙을 위한 Mashup 개발자 과정 (쌍용교육센터)
2016.07 기타
- 교육 항목 : Spring Framework, Oracle, javascript, jQuery, xml, Ajax
- 1번의 팀 프로젝트 경험(식당 예약 서비스)과 1번의 개인 프로젝트 경험 (쇼핑몰 제작)
- 식당 예약 서비스에서는 DB 설계 및 제작과 클라이언트의 식당 예약 보드 기획 및 개발을 맡았습니다.

## 링크
깃허브
https://github.com/Jin-dev92
기술적 고민을 적어 놓는 기술 블로그
https://hallowed-swoop-8ac.notion.site/265d99393f228010b0b4e450e77e4a0b?v=265d99393f2280a480bb000c2a0874b9

---

## (위키 메타) 요약
스타트업·프리랜서를 거친 5년 차 풀스택 개발자. 백엔드부터 인프라까지 직접 담당하며,
"느린 건 빠르게(API 응답 93% 단축), 없는 건 만들고, 반복되는 건 자동화(PR 파이프라인)"
하는 방식으로 일한다. NestJS/Spring Boot 백엔드와 React/Next.js 프론트, AWS 인프라가 주력.

## (위키 메타) 추출한 영구노트
- [[dev-profile-kim-uijin]]
