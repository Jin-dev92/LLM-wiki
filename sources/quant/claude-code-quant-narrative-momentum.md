---
type: source
title: 2026년 가장 잘 맞는 주식 매매법 — feat 클로드코드 퀀트매매
summary: 오드연구소(Hodu's AI Analysis Lab)가 Claude Code로 만든 퀀트 매매 스킬들의 주간 성과를 비교하고, 뉴스→종목 점수화 '네러티브 모멘텀' 전략을 공유.
created: 2026-06-25
updated: 2026-06-25
visibility: private
url: https://www.youtube.com/watch?v=luZQNhehDXU
author: Hodu's AI Analysis Lab (오드연구소)
ingested_via: youtube
tags: [quant, trading-strategy, claude-code, momentum, news]
---

## 요약
Claude Code로 만든 여러 퀀트 매매 "스킬"의 위크6~ 주간 수익률을 비교하며, 단일 전략 의존의 위험(레짐별 낙폭)과 코스피·미장의 변동성 차이를 짚는다. 그중 오늘 공유하는 전략은 **네러티브 모멘텀** — 뉴스/공시를 서사→테마→종목 점수로 변환(6단계, 누적·승수 스코어링)해 재료 직후 모멘텀을 포착한다. 전략 전문은 Notion에 공개되어 그대로 Claude Code에 이식 가능하다고 안내. 길이 5:43, 한국어.

## 추출한 영구노트
- [[narrative-momentum-strategy]]
- [[multi-strategy-regime-diversification]]

## 출처 원문 메모
- 전사: youtube 한국어 자동자막(`ko`) 기반 정제본. 영상은 화면 표(주간 손익·점수 예시)를 가리키며 설명하는 형식.
- 전략 목록 언급: VCP, 플로우 모멘텀 수급, 네러티브 모멘텀.
- 네러티브 모멘텀 6단계: 뉴스 수집 → 서사 점수화 → 테마 결정·영문섹터 한글번역 → 테마 종목 점수분배 → 상위 추천 → 매일 기록·누적.
- 스코어 예시(2026-06-12): 지정학 에너지 리스크 호재 133점, 섹터 로테이션 54점, 무역 리스크 악재 -28점 등 합 ~146점 → 신재생에너지(HD현대에너지솔루션·CS윈드·한솔 등) 추천. 점수는 반복 시 곱으로 누적.
- 적용 안내: Notion 전략 문서를 복사해 바이브 코딩에 투입하거나 기존 파이프라인에 적용.
