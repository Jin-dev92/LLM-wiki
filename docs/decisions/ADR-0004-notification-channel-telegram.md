# ADR-0004: 트레이딩 봇 알림 채널을 Telegram 봇으로 채택

## Status
Accepted

## Date
2026-06-21

## Context
quant-server(개인 알고리즘 트레이딩 봇)의 운영 관측 수단을 정한다. 봇은 트래픽 없이 24/7 자동
실행되므로, "보고 있지 않을 때 문제를 알려주는" **push형 알림**이 핵심이다(체결·에러·일일 PnL).
[[ADR-0003-algo-trading-phase0-design]]에서 관측은 자체 Grafana 없이 알림 + Sentry(무료) + CloudWatch 최소로 방향을 잡았고,
그중 **알림 채널**을 Slack / Discord / Telegram 중에서 선택한다. 비용 민감(프리티어 $0 지향),
보안 폐쇄(개인용), 한국 환경, 그리고 돈을 다루는 봇이라는 점이 판단 기준.

## Decision
**Telegram 봇**을 알림 채널로 채택한다.
- 봇이 **나에게 DM**으로 전송 → 서버/워크스페이스 불필요(가장 폐쇄적), 모바일 푸시가 가장 안정적.
- 발송: Bot API `sendMessage`(`chat_id`, `text`). 봇 토큰·`chat_id`는 **SSM Parameter Store(SecureString)**
  에 저장하고 EC2 인스턴스 롤로 읽어 주입한다(코드/레포/이미지/상태에 평문 없음 — 기존 시크릿 패턴).
- **양방향 확장 여지**: 추후 채팅 명령(`/status`, `/pause`, **`/kill` 긴급정지**)으로 봇을 제어 가능 →
  돈 다루는 봇의 안전장치로 가치가 큼.
- 구현은 Phase 2(알림 모듈). 본 ADR은 채널 선택만 확정.

## Alternatives Considered
### Discord (웹훅)
- Pros: 셋업 가장 쉬움(채널 웹훅 URL에 JSON POST), Embed 포맷 풍부.
- Cons: 서버 필요(비공개 가능하나 DM 대비 번거로움), 양방향은 별도 봇 추가 필요.
- Rejected: 단방향만이면 최선이나, 양방향(킬스위치) 확장성과 "DM 단독" 폐쇄성에서 Telegram에 밀림. (단방향만 원할 경우의 차선책)

### Slack (Incoming Webhook)
- Pros: 풍부한 Block 포맷, 팀 협업.
- Cons: 워크스페이스 필요, 무료 플랜 푸시 지연 가능, 슬래시커맨드 설정 복잡.
- Rejected: 이미 Slack에 상주하는 경우가 아니면 개인 봇 신규 도입엔 과함.

## Consequences
- Phase 2 알림 모듈은 Telegram Bot API에 의존. 토큰/chat_id 파라미터 2개를 SSM에 추가(`/quant-server/...`).
- 토큰 유출 시 @BotFather에서 즉시 재발급 가능(회전 용이).
- 양방향 명령 도입 시(추후): 명령 인증(내 chat_id만 허용)·killswitch는 [[ADR-0003-algo-trading-phase0-design]]의 리스크 관리(Phase 1)와 연계해 별도 설계.
- 적용 대상: `quant-server`의 README 로드맵(Phase 2)에 반영됨.

## 근거 자료
- 선행 결정: [[ADR-0003-algo-trading-phase0-design]] (관측 방향: 알림 + Sentry + CloudWatch 최소)
- 프로젝트 로드맵: `quant-server/README.md` (Phase 2)
