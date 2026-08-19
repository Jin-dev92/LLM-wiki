---
type: project
title: Day-1 부트스트랩 (새 PC 단일 파일 세팅 가이드)
summary: 새 회사 PC를 받은 날 순서대로 따라가면 Claude Code 작업 환경이 복원되는 실행 가이드 — 정책 확인, 전제 도구, 플러그인 3종(superpowers 중심)·스킬·훅·rtk 설치, 회사용 글로벌 CLAUDE.md 전문, 개인 위키 클론, 검증 체크리스트.
created: 2026-08-19
updated: 2026-08-19
visibility: private
status: active
tags: [meta, claude-code, onboarding, setup, bootstrap]
---

# Day-1 부트스트랩

> **사용법**: 새 PC에 Claude Code를 설치한 뒤, 이 파일 하나를 작업 폴더에 두고
> Claude Code 세션에서 이렇게 지시한다.
>
> ```
> day1-bootstrap.md 를 읽고 Step 0부터 순서대로 실행해줘.
> Step 0의 "입사 후 확인" 2건은 나한테 물어보고 넘어가.
> ```
>
> 환경 복원(Step 1~8)은 이 문서만으로 끝난다. 개인 위키(llm-wiki)를 회사 PC에 들여올
> 필요가 없다. 룰과 일부 스킬만 파일 번들로 반입한다(Step 9, 아래 준비물 참고).

**대상 환경**: macOS + zsh. 리눅스면 Homebrew 부분만 패키지 매니저에 맞게 바꾼다.
**성격**: 전부 멱등하다. 중간에 실패해도 처음부터 다시 돌려도 된다.

### 이 문서가 전제하는 것

들고 올 파일은 없다. **위키 레포를 회사 PC에 클론**해서 룰을 가져온다(Step 9).
필요한 건 개인 GitHub 계정 인증 하나뿐이다.

| 필요한 것 | 이유 |
|---|---|
| 이 문서 (`day1-bootstrap.md`) | 실행 순서 |
| 개인 GitHub 인증 | `Jin-dev92/LLM-wiki`가 **private** 레포라 clone에 인증이 필요하다 |

> 전제: **지급 PC를 개인 용도로도 쓸 수 있다**는 것. 회사 정책이 개인 계정 사용을 막으면
> Step 9만 불가능해지고, Step 1~8은 그대로 진행된다.

---

## Step 0 — 회사 정책 확인 (2026-08-19 기준 대부분 확정)

입사 전에 6개 중 4개가 확정됐다. **남은 2개만 입사 후 확인**하면 되고, 그중 하나만 실제로
설치 목록을 바꾼다.

### 확정 — 그대로 진행

| 항목 | 결론 | 영향 |
|---|---|---|
| 사내 코드의 외부 모델 전송 | **허용.** Claude Code 사용 가능 | 이 문서 전체 진행 가능 |
| 감사 로그 로컬 저장 | **개인 판단으로 설치**한다 | Step 6 진행. 로그는 로컬에만 두고 공유·커밋하지 않는다 |
| 사내 코드의 개인 레포 유출 | **회사용 워크스페이스 사용**으로 해결 | Step 1의 `gh auth`를 회사 계정으로 맞추기만 하면 됨 |
| 외부 MCP 커넥터 | **허용될 것으로 예상** | Step 10 진행. 막히면 건너뛰어도 개발 작업엔 지장 없음 |

### 입사 후 확인 — 2건

- [ ] **외부 네트워크 아웃바운드 허용 범위** ⚠️ *유일하게 설치 목록을 바꾸는 항목*
  - 사내망이 화이트리스트 방식이면 매 실행 외부 fetch가 발생하는 스킬이 실패하거나 보안 알람을 띄운다.
  - **막혀 있으면**: Step 4에서 `last30days`를 뺀다. 나머지는 로컬에서 돌아 영향 없다.
  - 확인처: 인프라/네트워크 담당, 또는 그냥 한 번 실행해보고 판단.

- [ ] **Claude 계정을 회사 계정으로 발급받기**
  - 개인 계정으로 사내 코드를 다루면 회사 자산이 개인 계정 대화 기록에 남는다(퇴사 후에도).
  - **회사 계정 발급을 먼저 요청**하고, 나오기 전까지는 사내 코드를 붙인 세션을 열지 않는다.
  - 요청처: 팀 리드 또는 IT/총무. "Claude 팀 플랜이 있는지"가 실질적인 질문이다.

> 계정 문제는 **설치 전에** 정리하는 게 싸다. 나중에 계정을 바꾸면 플러그인·스킬은 남지만
> 세션 기록과 연동은 새로 시작해야 한다.

---

## Step 1 — 전제 도구

```sh
# Homebrew가 없으면 먼저 설치
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

brew install git node python3 gh yt-dlp jq
```

**Orca 앱을 설치한다.** 워크트리·터미널 관리를 Orca로 하고, 작업 루트도 Orca 구조를 따른다.
앱을 깔면 `orca` CLI가 `/usr/local/bin/orca` → 앱 번들로 심볼릭 링크되므로 별도 설치가 없다.

```sh
orca --version                 # 앱 설치 후 CLI가 잡히는지 확인
mkdir -p ~/orca/workspaces     # 작업 루트 (앱이 안 만들어 놨다면)
```

> **작업 루트는 개인 PC와 동일하게 `~/orca/workspaces/`를 쓴다.** 클론한 레포
> (`llm-wiki`·`agent-skills`·`cc-audit-hooks`)가 여기 들어간다.
>
> 경로를 개인 PC와 맞추면 두 기기에서 같은 명령·같은 습관이 그대로 통하고, Step 8의 글로벌
> CLAUDE.md "개인 위키 참조" 경로도 개인 PC 버전과 동일해진다. 경로를 바꾸려면 이 문서와
> **Step 8 블록 안 경로를 함께** 고친다(두 곳이 맞아야 한다).

- `yt-dlp` — 유튜브 자막 추출용(과거 `watch` 플러그인을 대체). 회사 PC에선 선택.
- `gh` — `gh auth login`으로 **회사 계정** 인증. `gh auth status`로 활성 계정을 반드시 확인한다(개인 계정이 활성이면 사내 코드가 개인 레포로 나갈 수 있다).
- Orca — 워크트리·터미널 오케스트레이션. Step 8의 글로벌 CLAUDE.md가 `orca-cli`·`orchestration` 스킬을 전제하므로 설치해 둔다.

**검증**: `git --version && node -v && python3 -V && gh --version && orca --version`

---

## Step 2 — 플러그인 3종 (Claude Code 슬래시 명령)

셸이 아니라 **Claude Code 세션 안에서** 실행한다. 버전은 고정하지 않고 항상 최신을 쓴다.

```text
/plugin marketplace add obra/superpowers-marketplace
/plugin marketplace add anthropics/claude-plugins-official
/plugin marketplace add epoko77-ai/im-not-ai

/plugin install superpowers@superpowers-marketplace
/plugin install claude-md-management@claude-plugins-official
/plugin install humanize-korean@im-not-ai
```

각 플러그인의 역할:

| 플러그인 | 역할 |
|---|---|
| **`superpowers`** | 작업 규율의 뼈대 — `brainstorming`·`writing-plans`·`executing-plans`·`test-driven-development`·`systematic-debugging`·`verification-before-completion`. **이 킷에서 가장 중요한 플러그인이다** |
| `claude-md-management` | CLAUDE.md 감사·개선(`revise-claude-md`) — 사내 컨벤션을 레포 CLAUDE.md에 반영할 때 |
| `humanize-korean` | 한글 산출물(PR 본문·스펙 문서)의 AI 티 제거 |

**superpowers 훅 권한** — 스킬이 트리거되지 않으면 세션 시작 훅의 실행 권한 문제다.

```sh
chmod +x ~/.claude/plugins/superpowers/hooks/session-start.sh
```

**검증**
- `cat ~/.claude/plugins/installed_plugins.json` 에 3개가 보인다
- 새 세션에서 `/brainstorming` 이 인식된다 (superpowers 정상)
- 인식 안 되면 세션 재시작 → 그래도 안 되면 위 `chmod` → `/plugin install superpowers@superpowers-marketplace --force`

---

## Step 3 — gstack 스킬 스위트

```sh
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup
```

**검증**: `/office-hours` 인식 + `cat ~/.claude/skills/gstack/VERSION`

---

## Step 4 — standalone 스킬 2종

gstack과 형제 디렉터리라 `/gstack-upgrade`가 건드리지 않는다. 갱신은 각 디렉터리에서 `git pull`.

```sh
git clone --depth 1 https://github.com/DietrichGebert/ponytail.git   ~/.claude/skills/ponytail
git clone --depth 1 https://github.com/mvanhorn/last30days-skill.git ~/.claude/skills/last30days
```

- `ponytail` — 과설계 방지(YAGNI·최소 diff 강제)
- `last30days` — 최근 30일 여론·리서치. ⚠️ 매 실행 외부 fetch → 사내망이 화이트리스트면 제외(Step 0 미확정 항목)

---

## Step 5 — cherry-pick 스킬 3종 + mattpocock 발췌

### 5-1. agent-skills cherry-pick (3종)

`addyosmani/agent-skills` 전체로 전환하지 않고 공백을 메우는 3개만 가져온다.

```sh
git clone --depth 1 https://github.com/addyosmani/agent-skills.git ~/orca/workspaces/agent-skills
for s in documentation-and-adrs doubt-driven-development source-driven-development; do
  cp -R "$HOME/work/agent-skills/skills/$s" ~/.claude/skills/
done
```

> 경로 `skills/<이름>`은 2026-08-19에 upstream에서 직접 확인했다. clone 본은 지우지 말고
> 남겨둔다 — 자동 동기화가 없어 수동 갱신할 때 다시 쓴다.

- `documentation-and-adrs` — ADR **작성** 기능(gstack `document-generate`는 읽기만)
- `doubt-driven-development` — 보안·민감 로직의 적대적 fresh-context 리뷰
- `source-driven-development` — 공식문서 grounding으로 할루시네이션 방지

**SDD 훅 배선** — `~/.claude/settings.json`의 `hooks`에 아래를 병합한다(기존 항목 보존).

```json
{
  "PreToolUse": [
    { "matcher": "WebFetch",
      "hooks": [{ "type": "command", "command": "bash ~/.claude/skills/source-driven-development/hooks/sdd-cache-pre.sh", "timeout": 10 }] }
  ],
  "PostToolUse": [
    { "matcher": "WebFetch",
      "hooks": [{ "type": "command", "command": "bash ~/.claude/skills/source-driven-development/hooks/sdd-cache-post.sh", "timeout": 10, "async": true }] }
  ]
}
```

### 5-2. mattpocock 발췌 4종

```sh
npx skills@latest add mattpocock/skills \
  --skill codebase-design --skill tdd --skill diagnosing-bugs --skill research \
  -g -a claude-code -y
```

> `code-review`는 설치하지 않는다 — Anthropic 공식 마켓플레이스의 동명 1st-party 스킬과 정확히 충돌한다.
> `tdd`·`diagnosing-bugs`는 `superpowers:test-driven-development`·`systematic-debugging`과 기능이 겹친다(이름 충돌은 없음).

---

## Step 6 — 감사 로깅 훅

레포는 **public**이라 회사 PC에서 개인 GitHub 인증 없이 clone된다.

```sh
git clone https://github.com/Jin-dev92/cc-audit-hooks.git ~/orca/workspaces/cc-audit-hooks
cd ~/orca/workspaces/cc-audit-hooks && python3 install.py
```

- `install.py`가 `~/.claude/hooks/audit/` 배치 + `settings.json`의 UserPromptSubmit/PreToolUse/PostToolUse에 **기존 보존하며** 병합 등록한다(5-1의 SDD 훅과 공존). 멱등.
- 로그: `~/.claude/logs/audit/*.jsonl`(0600) + `index.db`. 리포트: `python3 ~/.claude/hooks/audit/audit_report.py`
- 차단 규칙: `~/.claude/hooks/audit/danger_rules.json`
- ⚠️ 레닥션은 휴리스틱이다. **로그 파일을 공유하거나 커밋하지 않는다.**

---

## Step 7 — rtk (Rust Token Killer) — ⚠️ Step 6 다음에 설치

셸 명령 출력을 LLM 컨텍스트 도달 전에 필터·압축하는 CLI 프록시. `ls`·`cat`·`grep`·`git diff`·
테스트 러너 등 100+ 명령에서 토큰을 60~90% 줄인다. 레거시 코드베이스를 대량으로 훑는
마이그레이션 작업에서 효과가 가장 크다.

```sh
brew install rtk
rtk init -g --no-patch     # settings.json을 자동 패치하지 말고 지시문만 출력시킨다
```

**`--no-patch`를 쓰는 이유 — 훅 순서 때문이다.** rtk는 PreToolUse에서 Bash 명령을
**재작성(rewrite)** 하고, cc-audit-hooks도 같은 PreToolUse에서 위험 명령을 **차단**한다.
rtk가 먼저 돌아 명령이 `rtk ...` 형태로 감싸지면 감사 훅의 위험 패턴 매칭이 빗나갈 수 있다.

- `~/.claude/settings.json`의 `PreToolUse` 배열에서 **audit 훅이 rtk 훅보다 앞에 오도록** 직접 배치한다.
- 이 시점의 `PreToolUse`에는 이미 셋이 등록돼 있다 — SDD(matcher `WebFetch`), audit(matcher 없음), Orca(matcher `*`). rtk를 넣으면 넷이 되므로 배열 순서를 눈으로 확인하고 넣는다.
- 출력만 압축하면 되므로, 가능하면 rtk 훅의 matcher를 `Bash`로 좁힌다.

**검증 (반드시 수행)** — 감사 훅의 차단이 rtk 설치 후에도 살아 있는지 확인한다.

```sh
python3 ~/.claude/hooks/audit/audit_report.py | tail -20
```

Claude Code 세션에서 `danger_rules.json`에 걸리는 명령(예: 재귀 강제 삭제)을 한 번 시도해
**차단되는지** 확인한다. 차단되지 않으면 rtk 훅을 `settings.json`에서 제거하고 재배치한다.

> 롤백: `cp ~/.claude/settings.json.bak ~/.claude/settings.json`

---

## Step 8 — 글로벌 CLAUDE.md 배치 (회사용 프로파일)

아래 블록을 그대로 `~/.claude/CLAUDE.md`에 쓴다. 개인 PC 버전에서 이력서 규칙·개인 위키
경로·Codex/GPT-5.6 디자인 레인을 걷어낸 회사용 파생본이다.

```sh
mkdir -p ~/.claude && cat > ~/.claude/CLAUDE.md <<'EOF_CLAUDEMD'
Always respond in Korean (한국어).

## 보안 원칙

### 1. RLS (Row Level Security)
- PostgreSQL 테이블 생성 시 RLS 활성화 여부를 반드시 확인한다
- 구독 상태(subscription)와 사용량 제한(rate_limit, api_usage)은 **반드시 별도 테이블로 분리**한다
- RLS 정책 작성 후, 반드시 "이 정책으로 다른 사용자 데이터에 접근 가능한 우회 경로가 있는가?"를 검토한다
- 테이블 설계 단계에서 민감 데이터와 사용량 데이터 혼재 여부를 사람이 직접 판단하도록 경고한다

### 2. Rate Limit
- 프론트엔드에만 rate limit을 두는 코드는 작성하지 않는다
- 반드시 **백엔드에서 사용자 ID + IP 기반 이중 제한**을 구현한다
- 사용량 과금 구조에서는 스팸 요청이 요금 폭탄으로 이어질 수 있음을 주석으로 명시한다

### 3. API 키 노출 방지
- AI API, 결제, 이메일, 클라우드 스토리지 등 민감한 외부 API는 **절대 프론트엔드에서 직접 호출하지 않는다**
- 환경변수(VITE_, NEXT_PUBLIC_ 등 클라이언트 노출 prefix)에 민감한 키를 넣지 않는다
- 민감한 API 호출은 반드시 백엔드 서버를 통한다
- AWS/GCP 등 클라우드 키 사용 시 예산 알림(Budget Alert) 설정을 코드 외부에서 반드시 하도록 안내한다

### 4. RBAC (역할 기반 액세스 제어) 필수 구현

### 반드시 지켜야 할 규칙
- 모르면 추측하지 말고 물어볼 것
- 파일을 삭제해야 하는 경우 물어보고 삭제 할 것
- context 가 70%를 초과한 경우 폴더 내 docs 폴더를 찾아 현재까지의 진행상황을 md 형식으로 저장하고 /clear 를 실행 후, 지금까지의 진행상황을 저장해둔 md 파일을 자동으로 읽도록 해
- PR 본문·작업 산출물에는 **항상 기획/계획이 되는 문서나 링크(스펙·플랜 md 경로, 이슈 URL 등)를 첨부**할 것. 리뷰어·미래의 내가 작업 근거와 맥락을 따라갈 수 있도록 한다
- PR을 작성하기 전에 **base 브랜치로부터 rebase 한 뒤 push**할 것. (예: `git fetch origin && git rebase origin/<base> && git push --force-with-lease`)

## 문서 작성 규칙 (PR 본문·스펙 문서)
- PR 본문과 스펙(설계) 문서는 **주니어 개발자가 읽어도 이해할 수 있는 수준**으로 쓴다. 배경 지식을 전제하지 말고, 왜 이렇게 했는지의 맥락을 담는다.
- 어투는 친근체가 아니라 **정보 전달 중심의 해설 격식체**를 쓴다.
  - 3인칭·개념 중심, `~입니다 / ~됩니다`체. 글쓴이("저는/우리 팀은")를 드러내지 않고 주제를 주어로 세운다.
  - "안녕하세요", `~이에요/해요`체, 이모지 남발, 수사적 질문 같은 구어체·친근체는 쓰지 않는다.
- 기술 용어는 **처음 등장 시 한 줄 설명**을 붙이고, 개념은 **점층적 복잡도**(기본→변형→고급)로 쌓으며, 추상적 흐름은 **번호 단계 + 다이어그램**으로 못박는다. **근거 없는 최상급**("최고의/완벽한/혁신적인")은 쓰지 않는다.

## AI 멀티에이전트 분업 정책
- 탐색·검색 → `explorer` 레인(haiku)
- 코드 구현·수정 → `coder` 레인(sonnet)
- 조율자는 설계 대화, 의사결정, 작업 분해 및 결과 종합을 담당한다.
- 사소한 인라인 편집, 대화형 답변은 조율자가 직접 처리할 수 있다.

### SDD ↔ orchestration 동시 사용 금지
`subagent-driven-development`(SDD)와 `orchestration`은 하나의 작업에 동시에 켜지 않는다.
섞으면 완료 신호와 진행 상태 저장소가 이중화된다.
- 같은 세션 안에서 계획을 실행 → **SDD**
- 다른 터미널·워크트리로 실제 위임, 소유권 이전 → **orca-cli**
- 위 위임에 감독·완료 대기·DAG·결정 게이트가 필요 → **orchestration**

## 개인 위키 참조 (llm-wiki)
- 경로: `~/orca/workspaces/llm-wiki`
- 축적 지식이 필요하면 파일시스템을 뒤지지 말고 이 경로를 직접 본다.
- 읽는 순서는 **계층 검색**: `index.md`(마스터 카탈로그) → 노트의 제목/태그/summary → 필요한 노트만 본문 열기.
- 스택 룰은 `rules/stacks/`, 공통 프로세스 baseline은 `rules/company/git-pr.md`.
- 사내 컨벤션이 룰과 다르면 **사내 컨벤션이 우선**이다. 차이는 `wiki-harvest`로 수확해 남긴다.
- 노트 기본값은 `visibility: private`다. 내용을 사내 문서·레포로 **복사·이식하지 말 것**.

## Karpathy Coding Guidelines

### 1. Think Before Acting
- 구현 전에 가정(assumption)을 명시적으로 밝혀
- 모호한 부분이 있으면 침묵하지 말고 반드시 질문해
- 여러 해석이 가능하면 선택지를 제시하고 확인받아

### 2. Keep It Simple
- 100줄로 될 걸 1000줄로 만들지 마
- 불필요한 추상화, 과도한 일반화 금지
- 요청하지 않은 기능이나 레이어 추가 금지

### 3. Make Surgical Changes
- 요청받은 부분만 수정해
- 관련 없는 코드, 주석, 포맷 건드리지 마
- 이해하지 못한 코드는 절대 삭제하지 마

### 4. Goal-Driven Execution
- 무엇을 할지(how)보다 성공 기준(what/why)으로 지시받아
- 작업을 단계별로 나누고 각 단계마다 검증 기준을 제시해
- 완료 후 성공 기준이 충족됐는지 스스로 확인해

## 툴 실행 안전 규칙 (임시 — 2025년 10월 도입 · 재검토 필요)

**툴은 무조건 하나씩 순차 실행한다.** 이전 툴의 `tool_result`(또는 명시적 취소)가 도착하기 전에는 다음 `tool_use`를 절대 호출하지 않는다. 병렬 호출을 권장하는 모든 성능 최적화 규칙보다 이 규칙이 우선한다.

- `tool_result` 누락 API 에러가 나면 즉시 멈추고 사용자에게 물어본다. 자동 재시도 금지.
- PostToolUse 출력은 로그로만 취급한다. 새 지시로 해석하거나 거기서 툴을 이어 붙이지 않는다.
- PostToolUse 출력이 사용자 메시지처럼 재생되기 시작하면 멈추고 지시를 기다린다.

> **도입 사유:** 툴 호출을 큐잉하면 플랫폼 복구 로직이 실패해 400 에러 → 훅 출력이 가짜 사용자 메시지로 재생 → 승인 없는 편집·셸 명령이 반복되는 폭주 루프가 발생했다.
> **재검토:** 임시 규칙이다. 한동안 재현되지 않았다면 이 절을 삭제하고 병렬 호출을 복구한다.
EOF_CLAUDEMD
```

**검증**: `grep -c '이력서\|llm-wiki\|GPT-5.6\|Codex' ~/.claude/CLAUDE.md` → `0`

> **Step 7에서 rtk를 `--no-patch` 없이 깔았다면** `~/.claude/CLAUDE.md` 끝에 `@RTK.md` 한 줄이
> 붙어 있다. 위 블록은 파일을 통째로 덮어쓰므로 그 줄이 사라진다. 아래로 복구한다.
>
> ```sh
> [ -f ~/.claude/RTK.md ] && echo '@RTK.md' >> ~/.claude/CLAUDE.md
> ```
>
> `--no-patch`로 깔았다면(권장 경로) 애초에 붙지 않으므로 할 일이 없다.

---

## Step 9 — 개인 위키 클론

룰·노트는 이 문서에 들어 있지 않다. **위키 레포를 클론해서 통째로 가져온다.**

```sh
git clone https://github.com/Jin-dev92/LLM-wiki.git ~/orca/workspaces/llm-wiki
```

`Jin-dev92/LLM-wiki`는 **private 레포**라 개인 GitHub 인증이 필요하다.
`gh auth login`이 회사 계정으로만 되어 있으면 clone이 실패한다 — `gh auth switch`로
개인 계정을 활성화하거나, 개인 계정용 read-only PAT로 clone한다.

> ⚠️ **회사 지급 PC는 결국 회사 자산이다.** 개인 용도 사용이 허용되더라도 반납·초기화
> 대상이고, MDM이 디스크를 백업할 수도 있다. 위키에는 이력서·개인 프로필 노트가
> `visibility: private`로 들어 있다. 그 사실을 알고 클론하는 것과 모르고 하는 것은 다르다.
>
> 신경 쓰이면 `sources/resume/`·`notes/dev-profile-*`만 빼고 쓰거나, 룰만 복사하는
> 방식으로 되돌리면 된다.

### 클론하면 따라오는 것

| 자산 | 위치 | 쓰임 |
|---|---|---|
| 스택 룰 전체 | `rules/stacks/`, `rules/company/` | 레포 CLAUDE.md 생성의 재료 (개수는 고정 표기하지 않는다 — 계속 늘어난다) |
| 위키 전용 슬래시 커맨드 | `.claude/commands/` | `ingest`·`wiki-apply`·`wiki-harvest`·`wiki-review` |
| 영구 노트 | `notes/` | 룰의 배경·이유 (`[[링크]]`로 연결돼 있다) |

레포에 `.claude/`가 포함돼 있어 **커맨드는 별도 설치 없이 따라온다.**

### 글로벌 CLAUDE.md의 경로를 맞춘다

Step 8에서 배치한 `~/.claude/CLAUDE.md`의 "팀 스택 룰 참조" 절이 이 경로를 가리켜야 한다.
Step 8 블록은 `~/orca/workspaces/llm-wiki` 기준으로 적혀 있으니, 다른 곳에 클론했다면 함께 고친다.

```sh
grep -n 'llm-wiki' ~/.claude/CLAUDE.md    # 경로가 실제 클론 위치와 같은지 확인
```

### 첫 레포에 CLAUDE.md 만들기

위키가 있으므로 `wiki-apply`를 그대로 쓴다.

```text
/wiki-apply <대상 프로젝트 경로>
```

사내 컨벤션이 룰과 다르면 **사내 쪽이 우선**이다. 차이를 발견하면 `wiki-harvest`로
사내 룰을 위키에 수확해 `rules/company/`에 새 파일로 남긴다
(`git-pr.md`는 employer-neutral baseline이므로 덮어쓰지 않는다).

---

## Step 10 — MCP 커넥터

- Notion / Gmail / Google Calendar / Google Drive — 각 도구 첫 호출 시 `authenticate` → `complete_authentication`
- 코드용: Playwright Test MCP — `npx playwright init-agents --loop=claude --prompts` (프로젝트 로컬 스코프라 레포마다 개별 실행)

> ⛔ **DB에 직접 붙는 MCP 커넥터는 연결하지 않는다.**
> DB 이관은 사람이 수행하기로 했고, MCP는 Claude에게 DB 직접 접근을 주는 유일한 경로다.
> 개인 PC 카탈로그(`claude-code-setup.md` §3·§6)에 AWS MFA + MySQL MCP 설정 가이드가
> 링크돼 있으나 **회사 PC에서는 설치 대상이 아니다.**

---

## Step 11 — 검증 체크리스트

```sh
cat ~/.claude/plugins/installed_plugins.json   # 플러그인 3개
cat ~/.claude/skills/gstack/VERSION            # gstack 설치 확인
ls ~/.claude/skills                            # ponytail·last30days·cherry-pick 3종 존재
rtk --version                                  # rtk 설치 확인
orca --version                                 # Orca CLI 확인
python3 -c "import json;print(list(json.load(open('$HOME/.claude/settings.json')).get('hooks',{}).keys()))"
ls ~/.claude/logs/audit/                       # 감사 로그 생성 여부
head -1 ~/.claude/CLAUDE.md                    # "Always respond in Korean"
ls ~/orca/workspaces/llm-wiki/rules/stacks/    # 스택 룰 (위키 클론 확인)
```

Claude Code 세션에서:
- [ ] `/office-hours` 인식 (gstack)
- [ ] `/brainstorming` 인식 (superpowers)
- [ ] 한국어로 응답
- [ ] 아무 Bash 명령 실행 후 `~/.claude/logs/audit/*.jsonl` 에 항목이 쌓인다
- [ ] **위험 명령이 여전히 차단된다** (rtk 훅이 감사 훅을 가리지 않았는지 확인 — Step 7)

---

## 부록 — 설치하지 않는 것과 이유

| 항목 | 이유 |
|---|---|
| `watch@claude-video` | 제거 확정(2026-08-19). 유튜브 자막은 `yt-dlp`로 대체 |
| `codex@openai-codex` | 외부 모델 = 정책 승인 대상. gstack의 `codex` 스킬로 대체 가능 |
| `taste-skill`, vercel 9종, `web-design-guidelines` | FE 디자인 계열 — 백엔드 포지션에서 사용처 없음 |
| `caveman` | 제외(2026-08-19). 응답 문체 압축은 회사 PC에서 불필요 |
| Serena MCP | 제외(2026-08-19). LSP 코드 인텔리전스 — 회사 PC 셋업에서 제외 |
| DB 직결 MCP 커넥터 | 제외. Claude에게 DB 직접 접근을 주는 경로라 "DB 이관은 사람이 수행" 원칙과 충돌한다 |

## 갱신 규칙
개인 PC에서 플러그인·스킬을 추가/제거하면 `guide/ai/claude-code-setup.md`(카탈로그)를 먼저
고치고, 회사 PC에 실제로 넣을 것만 이 문서에 반영한다. 명령어를 두 문서에 복붙하지 않는다 —
카탈로그는 "무엇이 있나", 이 문서는 "무엇을 어떤 순서로".
