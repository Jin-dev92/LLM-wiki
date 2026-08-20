# 신규 회사 온보딩 킷 (Day-1 부팅 + NestJS/Postgres 마이그레이션 준비)

**Goal:** 2026-09 신규 회사 입사 시 개인 PC를 수령한 당일에, 회사 정책 게이트를 통과한
범위 안에서 Claude Code 작업 환경 + 백엔드 스택 룰을 한 번에 복원한다. 타깃 스택은
NestJS + PostgreSQL(레거시 Express + MySQL 마이그레이션).

**Context:**
- 기존 초안 = `guide/ai/claude-code-setup.md`(환경 복원 카탈로그) + `guide/ai/audit-logging-setup.md`
- 타깃 스택 룰은 이미 존재 — `rules/stacks/{nestjs,prisma,nestjs-test,redis,resilience}.md`.
  `prisma.md`가 이미 PostgreSQL 기준이라 그대로 재사용 가능.

**확정된 결정 (2026-08-19):**
1. ~~회사 PC엔 위키를 통째 클론하지 않는다~~ → **2026-08-19 번복: 위키 레포를 회사 PC에 클론한다.**
   근거: 지급 PC를 개인 용도로도 쓸 수 있다는 전제가 확인됐다. USB·클라우드·별도 번들을
   모두 없애고 `git clone` 한 줄로 끝난다. `.claude/commands/`가 따라와 `wiki-apply`·
   `wiki-harvest`도 그대로 쓸 수 있다.
   남는 리스크(문서에 명시): 지급 PC는 회사 자산이라 반납·초기화 대상이고 MDM 백업 가능성이
   있는데, 위키에 이력서·개인 프로필 노트가 들어 있다. 신경 쓰이면 해당 폴더만 빼면 된다.
2. 회사용 글로벌 CLAUDE.md에서 **Codex/GPT-5.6 멀티에이전트 레인·FE 디자인 레인은 제거**한다.
   백엔드 포지션이라 사용처가 없고, 외부 모델 사용은 별도 승인 대상이다.
3. **cc-audit-hooks는 설치**한다. 위험 명령 차단 + AI 사용 통제 근거. 단 로그 보존 경로·기간을
   Day-1 정책 확인에 포함한다.

**정책 확인 결과 (2026-08-19, 사용자 확인):**
- Claude Code 사용 **가능** / 감사 훅은 **개인 판단으로 설치** / 회사용 워크스페이스 사용으로 개인 레포 유출 리스크 해소 / MCP는 허용 예상
- 입사 후 확인 2건: **외부 네트워크 아웃바운드**(막히면 `last30days` 제외), **회사 Claude 계정 발급 요청**

**작업 경계 (2026-08-19 사용자 확인):** **DB 이관은 AI를 쓰지 않고 사람이 직접 수행한다.**
스키마·데이터 이전, 컷오버, 시퀀스 보정은 사람 작업이다. AI는 이관 *이후*의 애플리케이션
코드와, 이관 시 무엇이 깨지는지 판단하는 데 쓰는 문서(`rules/stacks/postgres.md`)까지만 맡는다.

**AI의 DB 접근은 읽기 전용 계정으로만 허용한다.** DB 직결 MCP 커넥터는 쓰되 전용 읽기 전용
계정(`SELECT`만, 가능하면 replica)으로 붙인다. 경계를 문서가 아니라 **계정 권한**으로 강제한다.

**보류(입사 후 결정):** ORM(Prisma vs TypeORM) — 회사 선택 확인 후 룰 확정. 그전까지 TypeORM 룰은 쓰지 않는다.

---

## Track 0 — 입사 전 (지금 ~ 2026-08-31): 부팅 킷 완성

### Task 0-0. 설치 목록 실측 동기화 — ✅ 완료 (2026-08-19)
문서(플러그인 4개)와 실제 설치(3개)가 어긋나 있어 `~/.claude` 실측으로 재조사했다.

- [x] `guide/ai/claude-code-setup.md` §1 플러그인 표 갱신 — **버전은 전부 `latest`로 표기**(gstack에 적용한 2026-07-06 정책을 플러그인에도 확대). 현재: `superpowers`, `claude-md-management`, `humanize-korean`
- [x] `watch@claude-video` **제거 확정** — 재설치하지 않는다. 유튜브 자막은 이미 설치된 `yt-dlp`(2026.06.09)로 대체
- [x] watch 의존 지점 3곳 수정 — `CLAUDE.md` / `AGENTS.md`의 COMPILE 1단계, `.claude/commands/ingest.md`의 youtube 분기 → `yt-dlp --write-auto-sub --skip-download` (자막 없으면 사용자에게 확인)
- [x] `codex@openai-codex` 미설치 상태 반영 — gstack의 `codex` 스킬로 대체됨을 명시
- [x] §5-4 신설 — 문서에 없던 git clone standalone 스킬 5종(`ponytail`·`caveman`·`last30days`·`taste-skill`·`rtk`) + 회사 PC 설치 여부
- [ ] `/plugin marketplace remove claude-video` — watch 마켓플레이스 등록 잔여분 정리(라이브 환경 변경이라 사용자 직접 실행)

**회사 PC 설치 목록 (Task 0-3 Step 2~3의 입력):**
- 플러그인: `superpowers`, `claude-md-management`, `humanize-korean` (모두 latest)
- gstack 전체 + cherry-pick 3종(`documentation-and-adrs`·`doubt-driven-development`·`source-driven-development`)
- standalone: `ponytail`, `caveman`, `last30days`
- mattpocock 발췌: `codebase-design`·`tdd`·`diagnosing-bugs`·`research`
- 제외: `taste-skill`·vercel 9종·`web-design-guidelines`(FE 계열, 백엔드 포지션), `watch`(제거 확정), `codex`(외부 모델 = 정책 승인 대상)
- 보류: `rtk`(Rust 빌드 필요, 용도 재확인)

> ⚠️ `last30days`는 매 실행 외부 fetch가 발생한다. Step 0 정책 게이트의 "외부 네트워크 아웃바운드" 확인 항목에 포함할 것.

### Task 0-1. 글로벌 CLAUDE.md 사본 재동기화 — ✅ 완료 (2026-08-19)
- [x] `cp ~/.claude/CLAUDE.md guide/ai/claude-global.md` (현재 사본은 stale: 원본 156줄 중 74줄, 문서작성 규칙·멀티에이전트 분업·FE 디자인 레인·SDD 금지 4개 섹션 누락)
- [x] `guide/README.md`의 "마지막 동기화" 날짜 갱신 → 2026-08-19
- **검증:** `diff <(grep '^#' guide/ai/claude-global.md) <(grep '^#' ~/.claude/CLAUDE.md)` 출력 없음

### Task 0-2. 회사용 글로벌 프로파일 파생 — ✅ 완료 (2026-08-19, `day1-bootstrap.md` Step 8에 전문 인라인)
- [ ] `guide/ai/claude-global-work.md` 생성 — 0-1의 사본에서 아래를 제거/치환
  - 제거: 이력서 작성·사본 규칙, `## AI 멀티에이전트 분업 정책`의 Codex/FE 디자인 레인, `개인 위키 참조(llm-wiki)` 절의 개인 경로
  - 유지: 보안 원칙 4종(RLS·Rate Limit·API 키·RBAC), 반드시 지켜야 할 규칙, 문서 작성 규칙, Karpathy 가이드라인, 툴 실행 안전 규칙
  - 치환: 위키 참조 → 회사 PC에 반입한 `rules/` export 경로
- [ ] 상단에 "원본은 `claude-global.md`, 이 파일은 회사 PC 배치용 파생본" 명시
- **검증:** `grep -c '이력서\|llm-wiki\|GPT-5.6\|Codex' guide/ai/claude-global-work.md` → 0

### Task 0-3. Day-1 부트스트랩 문서 — ✅ 완료 (2026-08-19)
- [x] `guide/ai/day1-bootstrap.md` 생성 — 기존 `claude-code-setup.md`가 "카탈로그"라면 이건 "실행 순서"
  - **Step 0 (게이트):** 회사 AI 도구 정책 확인 — 사내 코드의 외부 모델 전송 허용 범위, Claude Code 계정(개인/회사), 감사 로그 보존 정책, 외부 MCP 커넥터 허용 여부. **통과 전 아무것도 설치하지 않는다.**
  - Step 1: Claude Code + CLI 도구 설치
  - Step 2: 플러그인 4종 + gstack (`claude-code-setup.md` §1~§2 참조 — 중복 서술 금지, 링크만)
  - Step 3: cherry-pick 스킬 3종 + 외부 스킬팩 (§5-1, §5-3)
  - Step 4: cc-audit-hooks 설치 (`audit-logging-setup.md`)
  - Step 5: `claude-global-work.md` → `~/.claude/CLAUDE.md` 배치
  - Step 6: rules export 반입 → 첫 레포에 `wiki-apply`로 CLAUDE.md 생성
  - Step 7: 레거시 코드베이스 파악 도구 선정(graphify vs gbrain 비교 — `claude-code-setup.md` §5에 이미 남긴 숙제)
- [~] `guide/ai/bootstrap.sh` — **작성하지 않음.** 문서 안의 셸 블록을 Claude가 순서대로 실행하는 방식이라 별도 스크립트는 중복이다. 필요해지면 그때 추출
- [x] rtk 포함 결정(사용자 요청) — Step 7 신설, 감사 훅과의 PreToolUse 순서 검증 절차 포함
- [x] ~~`guide/ai/bootstrap.sh`~~ — 자동화 가능한 부분만: gstack clone+setup, cc-audit-hooks clone+install.py, `npx skills` 2줄. `/plugin` 슬래시 명령은 스크립트화 불가 → 문서에만 남기고 스크립트는 안내 출력으로 끝낸다. 멱등하게.
- **검증:** 문서만 보고 순서대로 따라갈 수 있는지 셀프 리허설(설치 명령을 실제 실행하지 않고 경로·명령 존재만 확인)

### Task 0-4. 스택 룰 갭 메우기 — ✅ 완료 (2026-08-19)
- [x] `rules/stacks/postgres.md` — MySQL 출신 관점의 차이 중심. 최소 범위로: 대소문자·식별자 인용, `AUTO_INCREMENT` vs `identity/serial`, `utf8mb4` vs 인코딩, 타입 매핑(`tinyint(1)`→`boolean`, `datetime`→`timestamptz`), `ON UPDATE CURRENT_TIMESTAMP` 부재, 인덱스(부분/표현식), 트랜잭션 격리 기본값 차이, upsert(`ON CONFLICT`). ORM 무관한 DB 레이어 룰만 — Prisma 세부는 `prisma.md` 링크.
- [x] `rules/stacks/express-legacy.md` — 레거시를 **읽고 이관하기 위한** 룰. 미들웨어 체인·에러 처리 관례 파악법, 라우트 인벤토리 만드는 법, 암묵적 인증/세션 처리 찾기, 이관 전 "건드리지 않는다" 원칙.
- [x] 두 파일 모두 `visibility: team`, `scope: stack`, frontmatter는 `templates/rule.md` 준수
- [x] `index.md` rules 섹션에 2줄 추가
- **검증:** `/wiki-review`로 dead-link·frontmatter 정합성 통과

### Task 0-5. 마이그레이션 사전 노트 — ✅ 완료 (2026-08-19)
- [x] `notes/strangler-fig-migration.md` — Express→NestJS 점진 전환 패턴(라우트 단위 프록시 컷오버, 세션/인증 공존, 듀얼 라이트). **결론을 단정하지 말고** 입사 후 확인할 질문 목록을 포함:
  트래픽 규모, 배포 단위, 다운타임 허용치, MySQL→PG 데이터 이전 방식(덤프 vs CDC), 롤백 기준
- [x] `provenance: inferred` 표기(실제 코드베이스 미확인)
- [x] 기존 노트와 4개 링크(express-legacy·postgres·retry-idempotency-and-backoff·spec-as-code): `[[resilience-policy-composition-order]]` 또는 `[[spec-as-code]]`
- [x] `index.md` notes 섹션 갱신
- **검증:** `/wiki-review` 통과 + 단정 표현 없는지 자기 검토

### Task 0-6. rules export 리허설 — ✅ 완료 (2026-08-19)
- [x] `visibility: team` + `scope: stack` 필터로 임시 디렉터리에 export — **스택 룰 11개**
- [x] 유출 검사 통과(개인 식별 정보 0건)
- [x] **발견 1**: `rules/company/git-pr.md`가 이전 직장 지문(티켓 프리픽스 `THUB-XXX`, Spotless, Liquibase/React 전용 just 명령, Java 한정 TDD 요건)을 담고 있었음 → **employer-neutral baseline으로 재작성**(자리표시자화 + "사내 룰 우선" 명시). 반출 대상에 포함으로 전환
- [x] **발견 2**: `rules/stacks/java.md`에 이전 동료의 GitHub username(`ttc-cbj`)이 예시로 박혀 있었음 → `{github-username}` 자리표시자로 치환
- [x] 최종 반출 대상: 스택 룰 11개 + `git-pr.md` = 12개
- [~] **이 리허설은 결과적으로 불필요해졌다** — 위키 전체를 클론하기로 하면서 선별 반출 자체가 사라졌다. 다만 검사 과정에서 잡아낸 유출 2건(`THUB-XXX`, `ttc-cbj`)은 그대로 유효한 수확이다.
- **검증:** `grep -ri '이력서\|kim-uijin\|jindevst\|/Users/' <export경로>` → 개인 식별 정보 0건

---

> **Track 0 완료 (2026-08-19).** 남은 수동 작업 1건: `/plugin marketplace remove claude-video`(라이브 환경 변경).

## Track 1 — Day 1~3 (PC 수령 후)

- [ ] Step 0 정책 게이트 통과 항목만 설치 (`day1-bootstrap.md` 순서대로)
- [ ] 레거시 코드베이스 그래프화 — graphify 또는 gbrain 중 1개 선택, 선택 근거를 ADR로 기록
- [ ] 사내 Git/PR/배포/코드리뷰 룰을 `wiki-harvest`로 수확 → `rules/company/`에 신규 파일(기존 `git-pr.md`는 이전 회사 룰이므로 덮어쓰지 말고 분리)
- [ ] 첫 레포에 `wiki-apply`로 CLAUDE.md 생성 → 실제와 다른 부분을 조정
- **검증:** Claude Code 세션에서 `/office-hours`(gstack)·`/brainstorming`(superpowers) 인식, 감사 로그 JSONL 생성 확인

## Track 2 — 마이그레이션 착수 후

- [ ] ADR-0005 ORM 선택(Prisma vs TypeORM) — 확정 후 `rules/stacks/`에 반영
- [ ] ADR-0006 전환 전략(스트랭글러 단위·컷오버 기준)
- [ ] ADR-0007 MySQL→PostgreSQL 데이터 이전 방식·롤백 기준 (**실행은 사람이 한다** — ADR은 결정 기록용)
- [ ] 실측 결과를 `rules/stacks/postgres.md`·`notes/strangler-fig-migration.md`에 역류 (`provenance: inferred` → `extracted` 승격)

---

## 하지 않는 것 (스코프 제외)

- 실제 마이그레이션 설계 — 코드베이스를 보기 전엔 추측이다. Track 0은 "준비"까지만.
- TypeORM 룰 작성 — ORM 미확정.
- 회사 PC용 별도 위키 구축 — rules export로 충분. 필요해지면 그때.
- 위키 전체를 회사 PC로 클론 — 결정 1에 따라 금지.
