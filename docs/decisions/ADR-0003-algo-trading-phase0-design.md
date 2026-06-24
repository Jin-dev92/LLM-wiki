# ADR-0003: algo-trading 서버 Phase 0 설계 결정 (아키텍처·백테스트·범위·전략)

## Status
Accepted — 본 ADR의 "아키텍처" 결정은 [ADR-0002](ADR-0002-quant-server-feature-based-fastapi.md)의
기능별 구조 결정을 이 도메인에 맞게 **refine**한다(uv/ruff/mypy/pytest 툴체인 결정은 그대로 유효).

## Date
2026-06-21

## Context
quant-server의 실제 목적이 **국내+해외 알고리즘 트레이딩 자동화 서버**로 구체화됨(별도 설계 문서
검토). 제1원칙은 **"백테스트와 실거래가 같은 전략 코드를 쓴다"**. 사용자는 주식 투자 초보이고,
개인 토스증권 API 환경(발급 대기 중, 미확정 다수). 토스 발급 전 단계(Phase 0)를 어떻게 시작할지
의사결정을 한 단계씩 진행했다.

## Decision

### D1. 아키텍처 — 포트 최소 + 역할별 구조
`DataProvider`/`OrderExecutor`/`Strategy`를 ABC/Protocol 인터페이스로 두고, DI로 백테스트↔실거래
구현을 교체한다. **완전 헥사고날(domain/application/infrastructure 엄격 계층 분리)은 비채택** —
현 규모엔 과함. 추상화는 "갈아끼우는 지점"에만. 문서의 `core/adapters/backtest/api` 역할별
레이아웃 유지. (이중 모드 요구가 인터페이스를 사실상 강제하므로 ADR-0002의 "기능별" 라벨을 이
도메인에 맞게 조정)

### D2. 백테스트 엔진 — 자체 최소 이벤트 루프 + quantstats
bar 단위로 `DataProvider → Strategy.generate_signal → OrderExecutor`를 도는 수십 줄 루프를 직접
구현. 백테스트=Mock executor, 실거래=동일 루프+토스 → "같은 코드" 원칙에 충실. 지표·리포트는
**quantstats**(수익률 시계열만 받아 tearsheet 생성)에 위임.
- **backtrader 비채택**: 유지보수 정체 + 자체 루프 소유가 포트 구조와 충돌.
- backtesting.py는 리포트가 자체 엔진에 묶여 "보조"로 부적합. vectorbt는 대량 파라미터 최적화 시에만 보조 검토.

### D3. Phase 0 범위 — 백테스트 파이프라인만
결과는 파일(CSV/parquet). **DB(PostgreSQL/TimescaleDB)는 Phase 2**, **FastAPI 대시보드는 Phase 1~2**로
미룸(Phase 0엔 실주문·실시간 상태가 없어 불필요). 기존 `health` 슬라이스는 그대로 둔다.

### D4. 1차 전략 — 이동평균 교차(MA Crossover)
조절 변수 2개로 가장 단순, 차트로 직관적 → **파이프라인 검증 목적**에 적합(수익성은 이 단계 관심사
아님). RSI 평균회귀는 2차.

### 정합
패키지 관리 **uv**, Python **3.13** 유지 — 설계 문서의 poetry/3.11+를 대체(ADR-0002와 일치).

## Alternatives Considered
- **D1**: 완전 헥사고날(과함) / 추상화 없이 구체구현(이중 모드 원칙 깨짐) — 둘 다 기각.
- **D2**: backtrader(루프 충돌·정체) / backtesting.py(엔진 소유로 보조 부적합) / vectorbt(벡터화 강제, 이벤트 모델과 상충) — 기각.
- **D3**: DB·FastAPI 선도입 — 백테스트 검증을 지연시켜 기각.
- **D4**: RSI(튜닝 변수 多) / 둘 다 구현(Phase 0 범위 팽창) — 기각.

## Consequences
- Phase 0 구현 플랜으로 직결: 디렉토리 셋업 → 포트 인터페이스 → pykrx/yfinance 어댑터 → MA 전략 → 자체 루프 → quantstats 리포트 E2E.
- **리스크 관리는 Phase 1로 예약**: `RiskManager` seam만 비워두고, 실주문 진입 전 킬 스위치·포지션 사이징·주문 가드레일을 별도 결정(실거래 최우선 안전장치).
- **주문 멱등성(client_order_id)·브로커 reconciliation·감사 로그**는 Phase 1 설계 항목으로 예약.
- 토스 API 미확정 사항(인증·국내외 엔드포인트·rate limit·응답 포맷)은 `adapters/toss/`에 격리, 발급 후 결정.

## 근거 자료
- 설계 문서: algo-trading-server-design (사용자 제공, Claude Code 인계용)
- 선행 결정: [[ADR-0002]] (기능별 구조 + uv 툴체인)
- 프로젝트 컨벤션: `quant-server/CLAUDE.md`
