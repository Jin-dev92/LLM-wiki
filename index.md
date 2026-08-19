---
type: moc
title: Index
summary: 위키의 모든 페이지를 한곳에 모은 마스터 카탈로그. /ingest 시 자동 갱신.
created: 2026-06-18
updated: 2026-08-19
visibility: private
tags: [index]
---

# Index — 마스터 카탈로그

`/ingest` 가 생성/병합 시 이 파일을 갱신한다. 연관성 검색은 여기와 각 노트의
제목/태그/summary를 먼저 훑는다(계층 검색).

## notes
- [[workflow-vs-agent]] — Workflow는 미리 정의한 흐름, Agent는 LLM이 동적 결정. 트레이드오프.
- [[agentic-workflow-patterns]] — Anthropic의 재사용 5패턴(Chaining·Routing·Parallel·Orchestrator·Evaluator).
- [[agentic-reasoning-design-patterns]] — Andrew Ng의 4패턴(Reflection·Tool use·Planning·Multi-agent).
- [[dev-profile-kim-uijin]] — 5년 차 풀스택 개발자 프로필(강점·주력 스택·대표 성과).
- [[resume-writing-principles]] — 경력직 이력서 작성 원칙(상단 임팩트·Before→After·문제→대처·기여도).
- [[ai-experience-on-resume]] — AI 경험은 "도구 사용"이 아니라 "프로세스 설계·통제"로 써라. 승인 게이트·역할 분리·로깅 3신호 + 복붙 문구.
- [[agent-skill-authoring]] — SKILL.md 작성 패턴(description=무엇+Use when, 계층 로딩, rigid/flexible).
- [[agent-skill-archetypes]] — 에이전트 스킬 4가지 아키타입(워크플로우/취향/도구어댑터/프레임워크) + 도입 판단 규칙.
- [[ui-ux-reference]] — 토스·당근·Linear·Stripe 등 11개 서비스 UI/UX를 모방용 패턴·디자인 토큰·안티패턴으로 정리.
- [[harness-engineering]] — 모델 가중치 외부의 모든 것(지침·도구·지식·검증)을 설계해 에이전트 실수 재발을 막는 일.
- [[self-improvement-loop]] — 가이드+센서로 에이전트가 스스로 검증·교정하는 회로. Evaluator-Optimizer의 결정론적 변형.
- [[spec-as-code]] — E2E 테스트 코드 = 읽으면 명세·돌리면 검증인 두 얼굴.
- [[playwright-e2e-for-agents]] — Playwright E2E를 에이전트 검증 센서로 쓰는 실전(Critical Flows·flaky 예방·모킹·prefill·Trace·3 에이전트).
- [[agentic-context-platform]] — 자산 메타데이터를 자산화해 사람·AI 에이전트에 동일 컨텍스트를 공급하는 플랫폼(OpenMetadata·lineage·SKILL).
- [[narrative-momentum-strategy]] — 뉴스/공시를 서사→테마→종목 점수(누적·승수)로 변환해 재료 직후 모멘텀을 잡는 퀀트 전략(6단계).
- [[multi-strategy-regime-diversification]] — 변동성 레짐마다 맞는 전략이 달라 단일 스킬 의존은 위험. 주간 전략별 손익 추적·교체.
- [[redis-cluster-crossslot-prevention]] — Redis Cluster 멀티키 연산은 해시태그({})로 슬롯을 고정해야 CROSSSLOT 에러를 피한다.
- [[redis-mandatory-ttl-and-cache-invalidation]] — 캐시 쓰기엔 항상 TTL(capped 리스트 포함), 원본 mutation 시엔 관련 캐시 키 DEL, 파생 카운터는 DB COUNT 폴백으로 재구축.
- [[redis-connection-management-via-di]] — Redis 연결은 서비스별 직접 관리 대신 PrismaService처럼 DI 싱글턴(RedisModule)으로. 단 pub/sub duplicate 커넥션은 OnModuleDestroy에서 직접 정리.
- [[retry-idempotency-and-backoff]] — 재시도는 멱등+일시적 오류에만(4xx 반려), 지수 백오프+Jitter, 전체 타임아웃 예산.
- [[bulkhead-semaphore-isolation]] — 불안정 의존성의 동시 실행 수를 상한으로 격리. Node는 세마포어 방식, 수치는 실측 기반.
- [[circuit-breaker-fail-fast]] — 계속 실패하는 의존성은 호출 차단(fail-fast). 상태 변화 로깅 필수, fallback 설계.
- [[resilience-policy-composition-order]] — Timeout→Retry→CB→Bulkhead 순 wrap 합성, 의존성별 정책 인스턴스 분리.
- [[keyword-vs-semantic-retrieval]] — BM25는 단어가 겹쳐야 점수가 난다(우회 표현엔 0점). 벡터가 7배 메우지만 실력 차가 크면 RRF 융합은 손해.
- [[eval-set-is-also-under-test]] — 평가셋도 검증 대상. 베끼기 방지 제약이 과교정을 일으켜 성능이 7분의 1로 측정되고 결론이 뒤집혔던 사례.
- [[rag-value-at-small-corpus]] — 전체가 컨텍스트에 들어가는 규모에선 RAG는 정확도(1문항)가 아니라 비용(19배)의 장치. 질문 빈도가 판단 기준.
- [[strangler-fig-migration]] — 레거시를 라우팅 계층에서 한 기능씩 신규로 옮겨 고사시키는 전환 패턴. 난점은 라우팅이 아니라 데이터 공존·컷오버 단위·롤백 기준.

## sources
- [[sources/llm-agents/building-effective-agents]] — Anthropic 실용 에이전트 구축 가이드 (web).
- [[sources/llm-agents/andrew-ng-agentic-workflows-sequoia]] — Andrew Ng Sequoia 강연, 4패턴+실험 (youtube).
- [[sources/claude-code/claude-code-core-tools-guide]] — Claude Code 추천 도구 6종 설치 가이드 (local).
- [[sources/claude-code/claude-code-gstack-superpowers-guide]] — GStack+Superpowers 워크플로우 가이드 (local).
- [[sources/claude-code/agent-skills-repos-analysis]] — 공개 스킬 레포 4종 분석·도입 판정 (memo).
- [[sources/claude-code/ai-job-assistant-mcp-playmcp]] — 스티브의 파도타기, Claude Desktop+PlayMCP로 사람인·카카오톡 MCP 붙여 AI 취업비서 만들기 (youtube).
- [[sources/llm-agents/playwright-e2e-harness-naverpay]] — 네이버페이 FE의 Playwright E2E 하네스 구축 발표 (youtube).
- [[sources/llm-agents/context-provider-naver-d2]] — NAVER 플레이스, 사람·AI Agent 통합 Context Provider 구축 발표 (youtube).
- [[sources/resume/resume-fullstack-2026]] — 김의진 이력서, 경력·성과·스택·자격증 (pdf).
- [[sources/career/resume-recruiter-guide]] — 채용담당자 관점 이력서 개선 가이드 (local).
- [[sources/career/ai-experience-on-resume-thinklighthouse]] — 생각등대, 이력서 AI 경험 표현 3원칙+복붙 문구 (youtube).
- [[sources/quant/claude-code-quant-narrative-momentum]] — 오드연구소, Claude Code 퀀트매매 스킬 비교 + 네러티브 모멘텀 전략 (youtube).
- [[sources/backend/cc-redis-rule]] — 사용자 작성 Claude Code용 NestJS+Redis Cluster 코딩 룰(CROSSSLOT·TTL·DI) (local).
- [[sources/backend/resilience-patterns-team-rules]] — 사용자 작성 장애 대응 팀룰(Retry·Bulkhead·CB, NestJS+cockatiel) + 반입 시 보강 (local).

## rules
- [[rules/company/git-pr]] — Git/PR + 공통 프로세스 baseline(브랜치 보호·커밋 타입·PR 체크리스트·문서위치·테스트 요건·태스크 러너). 조직 고유 값은 자리표시자, 사내 룰 우선 (company).
- [[rules/stacks/java]] — Java/Spring 백엔드 규칙(QueryDSL·DTO·DDD·Liquibase 등) (stack).
- [[rules/stacks/nestjs]] — NestJS 일반 규칙(모듈·검증·설정키·트랜잭션, Prisma 기준) (stack).
- [[rules/stacks/prisma]] — Prisma DB 규칙(cuid·논리삭제·마이그레이션) (stack).
- [[rules/stacks/redis]] — Redis Cluster 캐시 규칙(해시태그로 CROSSSLOT 방지·TTL 필수·파생 카운터 DB 폴백·DI 연결관리+pub/sub 정리) (stack).
- [[rules/stacks/resilience]] — 장애 대응 패턴 규칙(멱등+일시 오류만 재시도·벌크헤드 격리·서킷 브레이커 로깅 필수·조합 순서, cockatiel/Resilience4j) (stack).
- [[rules/stacks/observability]] — APM 관측 규칙(증설·축소 판단 P0 계측 8종·스케일 방향 결정 트리·부하 테스트 기준선·scale-in 규칙 + 트레이싱/프로파일링 진단 계층). Node.js+PostgreSQL+ECS 예시 (stack).
- [[rules/stacks/postgres]] — PostgreSQL 규칙, MySQL 이관 관점(식별자 소문자 접힘·타입 매핑·identity 시퀀스·자동 갱신 컬럼 부재·upsert·격리수준 차이) (stack).
- [[rules/stacks/express-legacy]] — Express 레거시 파악·이관 규칙(읽기 전용 원칙·라우트 인벤토리·미들웨어 체인·이관 단위 판정) (stack).
- [[rules/stacks/nestjs-test]] — NestJS 테스트 코드 규칙 (stack).
- [[rules/stacks/frontend]] — React Query+Zustand+TS FE 개발 규칙 (stack).
- [[rules/stacks/react]] — React 함수형 컴포넌트 베이스 룰 (stack).
- [[rules/stacks/nextjs]] — Next.js App Router 규칙(Server/Client 경계·Server Action·NEXT_PUBLIC_ 민감키 금지) (stack).

## projects
- [[projects/estate-server]] — NestJS+Prisma+Kafka 플랫폼 백엔드 (최신 컨벤션 기준).
- [[projects/fanddle-server]] — Fanddle 플랫폼 백엔드 (Express+Sequelize 모노레포).
- [[projects/rag-wiki]] — 이 위키를 코퍼스로 RAG를 직접 구현하며 학습. 평가셋 우선, 6계단 벤치마크 기록.

## guide
- [[guide/ai/day1-bootstrap]] — 새 PC에 파일 하나만 던지면 끝나는 Day-1 세팅 가이드(정책 게이트→도구→플러그인·스킬·훅·rtk→회사용 글로벌 CLAUDE.md 전문→검증).
- [[guide/ai/claude-code-setup]] — Claude Code 환경 온보딩(플러그인·gstack 스킬·MCP·추천도구) + 동기화.
- [[guide/ai/audit-logging-setup]] — 감사 로깅 훅 cc-audit-hooks 복원(clone→install.py→검증→리포트).
- `guide/ai/claude-global.md` — 글로벌 `~/.claude/CLAUDE.md` 사본(원문 보관).
- `guide/mcp/` — Serena · AWS MFA+MySQL MCP 설정 가이드(원문 보관).
- `guide/aws/` — RDS 스냅샷 → 로컬 MySQL 복구 가이드(원문 보관).
- `guide/github-actions/` — Claude PR 자동 리뷰 워크플로우 설정·요약(원문 보관).

## decisions (ADR)
- `docs/decisions/ADR-0001-keep-gstack-superpowers-over-agent-skills.md` — Claude Code 기반을 gstack+superpowers로 유지(agent-skills 전환 안 함) + 3종 cherry-pick. flip condition 포함.
- `docs/decisions/ADR-0002-quant-server-feature-based-fastapi.md` — quant-server를 기능별(feature-based) FastAPI 구조로 채택(DDD 비채택) + uv/ruff/mypy/pytest 툴체인. 재검토 조건 포함.
- `docs/decisions/ADR-0003-algo-trading-phase0-design.md` — algo-trading 서버 Phase 0 결정: 포트 최소+역할별 구조(ADR-0002 refine)·자체 백테스트 루프+quantstats·백테스트만(DB/서버 보류)·MA 전략. 리스크관리는 Phase 1 예약.
- `docs/decisions/ADR-0004-notification-channel-telegram.md` — 트레이딩 봇 알림 채널을 Telegram 봇으로 채택(DM·양방향 킬스위치 확장성). Discord/Slack 비교. 토큰은 SSM SecureString.

## MOC
- [[MOC/Home]] — 위키 진입점.
- [[MOC/harness-engineering]] — 하네스 엔지니어링 주제 허브(하네스·자가개선 루프·E2E 센서).
