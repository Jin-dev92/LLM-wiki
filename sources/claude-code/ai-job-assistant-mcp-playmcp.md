---
type: source
title: 나만의 AI 취업비서 만들기 — Claude Desktop + PlayMCP(사람인·카카오톡 MCP)
summary: 스티브의 파도타기가 Claude Desktop에 PlayMCP 마켓플레이스로 사람인·카카오톡 MCP를 붙여 공고 수집→이력서 기준 등급 분류→Next.js 취업 상황판→기업 심층조사→이력서 첨삭까지 자동화하는 과정. 핵심은 사용 MCP 3종 정리.
created: 2026-06-30
updated: 2026-06-30
visibility: private
url: https://www.youtube.com/watch?v=QLfv4WovT74
author: 스티브의 파도타기 | AI 시대 일하는 법
ingested_via: youtube
tags: [mcp, claude-desktop, playmcp, saramin, job-search, agent]
---

## 요약
Claude Desktop에 **PlayMCP**(MCP 마켓플레이스)로 **사람인 MCP**와 **카카오톡 "나에게 보내기" MCP**를 연결해 "AI 취업비서"를 만든다. 흐름: 공고 자동 수집 → 내 이력서(`cv.md`) 기준 가중치 평가·A~D 등급 분류(`ranked-jobs.json`) → Next.js(App Router)+TS 취업 상황판 대시보드 → 사람인 `search_company_info`로 기업 심층조사 → 공고별 이력서 맞춤 첨삭. 길이 22:22, 한국어.

> ⚠️ 자막 추출 실패(유튜브 자막 429 / Whisper SSL 오류)로 **화면 프레임 기반** 정리. 도구함(03:21)·커넥터(16:13) 화면 근거.

## 사용 MCP (핵심)

### 1. PlayMCP (플랫폼/허브) — `playmcp.kakao.com`
MCP 자체가 아니라 **MCP를 검색·추가·배포하는 마켓플레이스 겸 커넥터 허브**. 도구를 켜고 Claude(또는 ChatGPT)에 연결한다. *(frame 02:48 / 03:21 / 03:55)*

### 2. 사람인(Saramin) MCP — 핵심
- 도구함 표기: **MCP Online, Tools 7개** *(frame 03:21)*
- 역할: 채용공고 검색·추천·회사 조사
- 커넥터에서 확인된 도구 이름 *(frame 16:13, 7개 중 6개 식별)*:

| 도구명 | 역할 |
|--------|------|
| `fetch_job_categories` | 직무 카테고리 조회 |
| `search_job_categories` | 직무 카테고리 검색 |
| `recommend_saramin_recruit` | 사람인 공고 추천 |
| `search_location_codes` | 지역 코드 검색 |
| `search_company_info` | 회사 정보 조회 (**심층조사에 사용**) |
| `search_subway_info` | 지하철(역) 정보 검색 |

### 3. 카카오톡 "나에게 보내기" MCP
- 도구함 표기: **MCP Online, Tools 1개** *(frame 03:21)*
- 역할: 공고 알림 등 원하는 정보를 카카오톡 "나에게 보내기"로 전송 (MemoChat 기반)

> MCP가 아닌 것: `github.com/santifer/career-ops`는 이력서 기반 공고 평가 프레임워크(오픈소스 프롬프트/로직 자산). 커넥터 사이드바엔 Context7·Gmail·Google Drive·Notion·Telegram 등도 보이나 **이 프로젝트 실사용은 사람인+카카오톡 2개**.

## 추출한 영구노트
- _(아직 없음 — 출처 메모로만 보관)_

## 출처 원문 메모 (진행 단계)
- **5대 기능**(frame 01:07): 공고 찾기 / 공고 저장 / 맞춤 분류 / 취업 상황판 / 카톡 알림.
- **환경**: Claude Desktop 로컬 프로젝트 `new_job`. PlayMCP 도구함에서 사람인·카카오톡 MCP 추가 후 Claude 커넥터로 연결.
- **공고 평가 가중치**(frame 07:16, 각 0~5점): 직무 적합도(JD 매핑)×3, 기술스택 일치(cv.md↔공고 키워드)×2, 경력연차 부합×1, 근무조건(지역/원격/연봉)×1, 성장성(회사 규모·도메인)×1.
- **산출 파일**: `cv.md`(career-ops 형식 이력서), `data/ranked-jobs.json`(공고 약 20개 랭킹: company·title·url·score·grade·reason·missing_keywords), `data/reports/<rec_idx>.json`(기업 심층조사 리포트).
- **대시보드**(frame 08:57~10:04): Next.js(App Router)+TypeScript, 정적 JSON fetch, DB 없음. localhost:3000 "취업 상황판" — 상단 요약(총/등급별/평균점수) + 공고 리스트(매칭이유·부족키워드·지원상태·보고서 버튼·원본링크).
- **심층조사**(frame 11:45): 사람인 `search_company_info`로 회사 조사 + JD↔cv.md 교차 분석 리포트 생성.
- **이력서 첨삭**(frame 12:52~13:59): 공고별 주입 키워드·추가 불릿·강조 불릿 제안, 우측 상세 패널에 회사 분석·면접 예상 질문 표시.
- (후반 14:00~22:22 구간 일부 미확인 — 면접 준비·카톡 알림 마무리.)
