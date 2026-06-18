# Claude PR Workflow 설정 가이드

> 새로운 프로젝트에 Claude Code를 활용한 자동 PR 리뷰 워크플로우를 구축하는 전체 절차

**작성일**: 2025-11-11
**프로젝트**: thub-fe-be
**참조 문서**: https://code.claude.com/docs/en/github-actions

---

## 📋 목차

1. [사전 요구사항](#사전-요구사항)
2. [Phase 1: Claude Code 인증 설정](#phase-1-claude-code-인증-설정)
3. [Phase 2: 프로젝트 문서 구성](#phase-2-프로젝트-문서-구성)
4. [Phase 3: GitHub Actions 워크플로우 설정](#phase-3-github-actions-워크플로우-설정)
5. [Phase 4: Notion MCP 통합](#phase-4-notion-mcp-통합)
6. [Phase 5: 테스트 및 검증](#phase-5-테스트-및-검증)
7. [참고 자료](#참고-자료)

---

## 사전 요구사항

### 권한 및 계정

- [ ] GitHub 저장소 관리자 권한
- [ ] Claude Pro 또는 Max 구독 (OAuth Token 생성용)
- [ ] Anthropic API 접근 권한 (또는 AWS Bedrock/Google Vertex AI)
- [ ] Notion Integration 생성 권한 (선택 사항)

### 로컬 환경

- [ ] Claude Code CLI 설치 완료
- [ ] Git 설정 완료
- [ ] Node.js 20+ 설치 (Notion MCP용)

---

## Phase 1: Claude Code 인증 설정

### 1.1 Claude Code OAuth Token 생성

**방법 A: Quick Setup (권장)**

```bash
# Claude Code CLI에서 실행
claude
# 프롬프트에서 입력
/install-github-app
```

이 명령어는 다음을 자동으로 처리합니다:
- GitHub 앱 설치
- OAuth Token 생성
- GitHub Secrets 설정 안내

---

**방법 B: Manual Setup**

1. **OAuth Token 생성**
   ```bash
   claude setup-token
   ```
   - Pro/Max 사용자만 가능
   - 생성된 토큰을 안전한 곳에 복사

2. **Claude GitHub 앱 설치**
   - https://github.com/apps/claude 접속
   - "Install" 클릭
   - 저장소 선택 (개별 또는 조직 전체)
   - 권한 승인:
     - Contents: Read & Write
     - Issues: Read & Write
     - Pull Requests: Read & Write

---

### 1.2 GitHub Secrets 설정

1. **저장소 설정 접근**
   - GitHub 저장소 → Settings 탭

2. **Secrets 추가**
   - Secrets and variables → Actions
   - "New repository secret" 클릭

3. **인증 토큰 추가**

   | Secret 이름 | 값 | 설명 |
   |-------------|-----|------|
   | `CLAUDE_CODE_OAUTH_TOKEN` | `claude setup-token` 출력값 | OAuth 인증 (Pro/Max 전용) |
   | `ANTHROPIC_API_KEY` | `sk-ant-api03-...` | API 키 인증 (대체 방안) |

   **주의:** 둘 중 하나만 설정하면 됩니다.

4. **저장 확인**
   - "Add secret" 클릭
   - Secrets 목록에서 이름만 표시되는지 확인 (값은 숨김)

---

### ✅ Phase 1 체크리스트

- [ ] `claude setup-token` 실행 완료 또는 `/install-github-app` 완료
- [ ] Claude GitHub 앱 설치 완료
- [ ] `CLAUDE_CODE_OAUTH_TOKEN` 또는 `ANTHROPIC_API_KEY` Secret 추가 완료
- [ ] Secret이 저장소 Settings → Actions에서 확인됨

---

## Phase 2: 프로젝트 문서 구성

### 2.1 프로젝트 컨텍스트 문서 작성

**docs/PROJECT_INSIGHT.md 생성**

```markdown
# [프로젝트명] 컨텍스트

## 프로젝트 구조
- 프로젝트 개요
- 디렉토리 구조
- 주요 모듈 설명

## 기술 스택
- 백엔드: Spring Boot, Gradle
- 데이터베이스: MySQL, Liquibase
- 인프라: AWS ECS, CloudWatch

## 개발 방법
- TDD/BDD 원칙
- 코딩 컨벤션
- Git 브랜치 전략

## 대화 규칙
1. 피드백은 기술 용어 제외 한글로 작성
2. 코드 스니펫 10줄 이내
3. 문자열 표기 시 "" 사용
4. Liquibase 마이그레이션 시 환경별 설정 준수
```

---

### 2.2 프로젝트 인덱스 문서 작성

**docs/PROJECT_INDEX.md 생성**

```markdown
# Project Index: [프로젝트명]

Generated: YYYY-MM-DD

## 📊 Project Overview
- Language: Java 17
- Framework: Spring Boot
- Build: Gradle
- Database: MySQL 8

## 📁 Project Structure
\```
project/
├── src/main/java/          # 소스 코드
├── src/test/java/          # 테스트
├── docs/                   # 문서
└── .github/workflows/      # CI/CD
\```

## 🏗️ Architecture
- Controller → Service → Repository 레이어 구조
- DTO vs Entity 분리
- 공통 모듈: common, util

## 🔑 Key Components
### Controllers
- UserController: 사용자 관리
- OrderController: 주문 처리

### Services
- UserService: 비즈니스 로직
- OrderService: 주문 로직

### Repositories
- JPA Repository 패턴 사용
```

---

**docs/PROJECT_INDEX.json 생성 (선택 사항)**

```json
{
  "project": "thub-fe-be",
  "version": "1.0.0",
  "modules": {
    "tatoa-webapp": {
      "type": "web",
      "controllers": ["UserController", "OrderController"],
      "services": ["UserService", "OrderService"]
    },
    "tatoa-common": {
      "type": "library",
      "entities": ["User", "Order"]
    }
  }
}
```

---

### ✅ Phase 2 체크리스트

- [ ] `docs/PROJECT_INSIGHT.md` 작성 완료
- [ ] `docs/PROJECT_INDEX.md` 작성 완료
- [ ] `docs/PROJECT_INDEX.json` 작성 완료 (선택)
- [ ] 프로젝트 구조와 기술 스택이 정확히 반영됨
- [ ] 개발 규칙 및 컨벤션이 명확히 기술됨

---

## Phase 3: GitHub Actions 워크플로우 설정

### 3.1 워크플로우 파일 생성

**.github/workflows/on-pr-claude-review.yml**

```yaml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize]
    branches:
      - dev
      - release
      - main
    paths:
      - "src/**/*.java"         # 백엔드 Java 파일
      - "liquibase/**/*.sql"    # DB 마이그레이션
      - "liquibase/**/*.yaml"

jobs:
  claude-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write  # write 필요 (gh pr comment)
      issues: read
      id-token: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for better context

      - name: Check for Notion URL in PR
        id: check-notion
        run: |
          PR_BODY=$(gh pr view ${{ github.event.pull_request.number }} --json body -q .body)
          if echo "$PR_BODY" | grep -qiE "notion\.so|notion\.site"; then
            echo "has_notion=true" >> $GITHUB_OUTPUT
          else
            echo "has_notion=false" >> $GITHUB_OUTPUT
          fi
        env:
          GH_TOKEN: ${{ github.token }}

      - name: Setup Node.js for Notion MCP
        if: steps.check-notion.outputs.has_notion == 'true'
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install Notion MCP Server
        if: steps.check-notion.outputs.has_notion == 'true'
        run: npm install -g @notionhq/notion-mcp-server

      - name: Run Claude Code Review
        id: claude-review
        uses: anthropics/claude-code-action@v1
        env:
          NOTION_API_KEY: ${{ secrets.NOTION_API_KEY }}
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          prompt: |
            REPO: ${{ github.repository }}
            PR NUMBER: ${{ github.event.pull_request.number }}

            ## 리뷰 가이드라인

            ### 필수 참조 문서
            다음 문서들을 읽고 프로젝트 아키텍처와 개발 규칙을 이해한 후 리뷰를 진행하세요:
            - **docs/PROJECT_INSIGHT.md** - 프로젝트 컨텍스트, 기술 스택, 대화 규칙
            - **docs/PROJECT_INDEX.md** - 프로젝트 구조 및 아키텍처
            - **docs/PROJECT_INDEX.json** - 상세 코드 매핑

            ### Notion 문서 참조
            PR description에 Notion 링크가 있다면 다음 방법으로 읽으세요:
            1. `gh pr view --json body`로 PR description 확인
            2. Notion 페이지 URL 추출 (예: https://notion.so/xxx)
            3. Notion MCP 도구를 사용하여 페이지 내용 읽기:
               - `mcp__notion__API-retrieve-a-page` (페이지 읽기)
               - `mcp__notion__API-get-block-children` (상세 블록 읽기)
            4. Notion 문서의 요구사항/컨텍스트를 반영하여 리뷰

            ### 리뷰 중점 항목
            1. **기능 정확성**
               - 요구사항 충족 여부
               - 비즈니스 로직 검증
               - 엣지 케이스 처리

            2. **버그 및 안정성**
               - Null 체크, Optional 사용
               - 예외 처리 (try-catch, @Transactional)
               - 리소스 관리 (DB 커넥션, 파일 핸들)
               - 동시성 이슈 (Race condition, Deadlock)

            3. **성능**
               - 쿼리 최적화 (N+1 문제, JOIN vs 여러 쿼리)
               - 인덱스 활용 여부
               - 불필요한 DB 호출 제거
               - 캐싱 전략 (Redis, @Cacheable)

            4. **테스트**
               - 단위 테스트 존재 여부
               - 테스트 커버리지 (중요 로직 80% 이상)
               - 엣지 케이스 테스트
               - Mock 사용의 적절성
               - TDD/BDD 원칙 준수

            5. **코드 품질**
               - 가독성 (메서드 길이, 변수명)
               - 중복 코드 제거 (DRY 원칙)
               - 복잡도 (순환 복잡도 10 이하)
               - 적절한 추상화 레벨
               - SOLID 원칙 준수

            6. **아키텍처**
               - 레이어 분리 (Controller → Service → Repository)
               - 단일 책임 원칙 (SRP)
               - 의존성 방향 (상위 → 하위)
               - DTO vs Entity 사용 적절성

            7. **문서화**
               - 복잡한 로직에 주석
               - API 문서 업데이트 (Swagger/OpenAPI)
               - README 업데이트 (필요 시)

            ### 실행 방법
            1. `gh pr view --json body,title`로 PR 정보 확인
            2. PR description에서 Notion 링크 추출 및 읽기 (있는 경우)
            3. `gh pr diff`로 변경 사항 확인
            4. docs/PROJECT_INSIGHT.md의 대화 규칙(특히 5번 한글 규칙)을 준수하여 리뷰 작성
            5. `gh pr comment`로 리뷰 코멘트 등록

          claude_args: '--allowed-tools "Bash(gh issue view:*),Bash(gh search:*),Bash(gh issue list:*),Bash(gh pr comment:*),Bash(gh pr diff:*),Bash(gh pr view:*),Bash(gh pr list:*),mcp__notion__API-retrieve-a-page,mcp__notion__API-get-block-children,mcp__notion__API-post-search"'
```

---

### 3.2 워크플로우 커스터마이징

**프로젝트별 수정 필요 항목:**

1. **파일 경로 필터** (line 10-13)
   ```yaml
   paths:
     - "your-module/**/*.java"
     - "database/**/*.sql"
   ```

2. **대상 브랜치** (line 6-9)
   ```yaml
   branches:
     - main
     - develop
   ```

3. **리뷰 항목 조정** (line 72-113)
   - 프로젝트 특성에 맞게 항목 추가/제거

---

### ✅ Phase 3 체크리스트

- [ ] `.github/workflows/on-pr-claude-review.yml` 생성 완료
- [ ] 파일 경로 필터가 프로젝트 구조에 맞게 설정됨
- [ ] 대상 브랜치가 올바르게 설정됨
- [ ] 리뷰 항목이 프로젝트 요구사항을 반영함
- [ ] `claude_code_oauth_token` Secret 참조가 정확함

---

## Phase 4: Notion MCP 통합

### 4.1 Notion Integration 생성

1. **Notion Integration 페이지 접속**
   - https://www.notion.so/my-integrations

2. **새 Integration 생성**
   - "New integration" 클릭
   - 이름: "Claude PR Review" (또는 원하는 이름)
   - Associated workspace 선택

3. **권한 설정**
   - **Capabilities:**
     - ✅ Read content
     - ⬜ Update content (불필요)
     - ⬜ Insert content (불필요)
   - **User capabilities:**
     - ⬜ Read user information without email
     - ⬜ Read user information with email

4. **Integration Token 복사**
   - "Submit" 클릭 후 표시되는 "Internal Integration Token" 복사
   - 형식: `secret_...`

---

### 4.2 Notion 페이지에 Integration 연결

리뷰할 때 참조할 Notion 페이지에 Integration을 연결해야 합니다:

1. **Notion 페이지 열기**
   - 참조할 요구사항/이슈 문서 페이지

2. **Connection 추가**
   - 페이지 우측 상단 ⋯ (더보기) 클릭
   - "Connections" 메뉴
   - "Claude PR Review" Integration 선택
   - "Confirm" 클릭

---

### 4.3 GitHub Secret 추가

1. **저장소 설정**
   - Settings → Secrets and variables → Actions

2. **새 Secret 추가**

   | Secret 이름 | 값 | 설명 |
   |-------------|-----|------|
   | `NOTION_API_KEY` | `secret_...` | Notion Integration Token |

3. **저장**
   - "Add secret" 클릭

---

### 4.4 로컬 MCP 설정 (선택 사항)

로컬에서 Claude Code를 사용할 때도 Notion MCP를 활성화하려면:

```bash
# Notion MCP 서버 설치
npm install -g @notionhq/notion-mcp-server

# MCP 설정
claude mcp add notion
# 프롬프트에서 NOTION_API_KEY 입력

# 연결 확인
claude mcp list
```

---

### ✅ Phase 4 체크리스트

- [ ] Notion Integration 생성 완료
- [ ] "Read content" 권한 활성화
- [ ] Internal Integration Token 복사
- [ ] 참조할 Notion 페이지에 Integration 연결
- [ ] `NOTION_API_KEY` GitHub Secret 추가 완료
- [ ] 로컬 MCP 설정 완료 (선택 사항)

---

## Phase 5: 테스트 및 검증

### 5.1 워크플로우 문법 검증

```bash
# GitHub CLI 설치 확인
gh --version

# 로컬에서 워크플로우 문법 검증 (act 도구)
brew install act  # macOS
act -l  # 워크플로우 목록 확인
```

---

### 5.2 테스트 PR 생성

1. **테스트 브랜치 생성**
   ```bash
   git checkout -b test/claude-pr-review
   ```

2. **간단한 변경 사항 추가**
   ```bash
   # 예: Java 파일에 주석 추가
   echo "// Test comment" >> src/main/java/com/example/Test.java
   git add .
   git commit -m "test: Claude PR review workflow"
   git push origin test/claude-pr-review
   ```

3. **PR 생성**
   ```bash
   gh pr create --title "test: Claude PR review workflow" \
                --body "Testing Claude Code PR review automation" \
                --base dev
   ```

   **Notion 링크 포함 테스트:**
   ```markdown
   ## 이슈
   https://www.notion.so/your-workspace/page-id

   ## 변경 사항
   - Claude PR review 워크플로우 테스트
   ```

---

### 5.3 워크플로우 실행 확인

1. **GitHub Actions 탭 확인**
   - 저장소 → Actions 탭
   - "Claude Code Review" 워크플로우 실행 확인

2. **로그 확인**
   - 실행 중인 워크플로우 클릭
   - "Run Claude Code Review" 단계 로그 확인
   - Notion 페이지 읽기 성공 여부 확인

3. **PR 코멘트 확인**
   - PR 페이지로 돌아가기
   - Claude가 남긴 리뷰 코멘트 확인
   - 한글로 작성되었는지 확인
   - 7개 카테고리가 반영되었는지 확인

---

### 5.4 문제 해결

**워크플로우가 실행되지 않는 경우:**

- [ ] `paths` 필터가 변경된 파일과 일치하는지 확인
- [ ] `branches` 설정이 PR의 base 브랜치와 일치하는지 확인
- [ ] GitHub Actions가 저장소에서 활성화되어 있는지 확인

**⚠️ 첫 워크플로우 추가 시 정상 경고:**

```
Warning: Skipping action due to workflow validation:
Workflow validation failed. The workflow file must exist and have
identical content to the version on the repository's default branch.
```

**이것은 정상입니다!**
- 새 워크플로우 파일을 PR로 추가할 때 항상 발생
- 보안상 PR의 워크플로우는 base 브랜치에 머지된 후에만 실행됨
- **해결 방법:**
  1. 현재 PR을 그대로 진행하여 dev/main에 머지
  2. 머지 후 새로운 테스트 PR 생성
  3. 이후 PR부터는 워크플로우가 정상 실행됨

**인증 오류 발생 시:**

- [ ] `CLAUDE_CODE_OAUTH_TOKEN` 또는 `ANTHROPIC_API_KEY`가 올바르게 설정되었는지 확인
- [ ] Secret 이름 철자가 정확한지 확인 (대소문자 구분)
- [ ] `claude setup-token`을 다시 실행하여 토큰 재생성

**Notion 연동 실패 시:**

- [ ] `NOTION_API_KEY`가 올바르게 설정되었는지 확인
- [ ] Notion 페이지에 Integration이 연결되었는지 확인
- [ ] Notion 페이지가 삭제되지 않았는지 확인
- [ ] Integration에 "Read content" 권한이 있는지 확인

---

### ✅ Phase 5 체크리스트

- [ ] 테스트 PR 생성 완료
- [ ] 워크플로우가 자동으로 실행됨
- [ ] Claude가 PR에 리뷰 코멘트를 남김
- [ ] 리뷰 코멘트가 한글로 작성됨
- [ ] Notion 링크가 있는 경우 내용이 반영됨
- [ ] 7개 카테고리 리뷰 항목이 확인됨

---

## 📊 필수 GitHub Secrets 요약

| Secret 이름 | 필수 여부 | 값 예시 | 발급 방법 | 설명 |
|-------------|----------|---------|-----------|------|
| `CLAUDE_CODE_OAUTH_TOKEN` | ✅ (또는 API 키) | `oauth_...` | `claude setup-token` | OAuth 인증 (Pro/Max) |
| `ANTHROPIC_API_KEY` | ⬜ (또는 OAuth) | `sk-ant-api03-...` | console.anthropic.com | API 키 인증 (대체) |
| `NOTION_API_KEY` | ⬜ (선택) | `secret_...` | notion.so/my-integrations | Notion 연동 |

**중요:**
- `CLAUDE_CODE_OAUTH_TOKEN`과 `ANTHROPIC_API_KEY` 중 **하나만** 설정하면 됩니다
- Pro/Max 사용자는 OAuth Token 사용 권장
- Notion 연동은 선택 사항 (PR description에 Notion 링크를 포함할 경우에만 필요)

---

## 🔗 참고 자료

### 공식 문서

1. **Claude Code GitHub Actions**
   - https://code.claude.com/docs/en/github-actions
   - Quick setup, Manual setup, Best practices

2. **Claude Code Action Repository**
   - https://github.com/anthropics/claude-code-action
   - Setup guide, Cloud providers, Migration guide

3. **Notion API Documentation**
   - https://developers.notion.com/
   - Integration 생성, 권한 설정, API 참조

### 내부 문서

- `docs/PROJECT_INSIGHT.md` - 프로젝트 컨텍스트 및 개발 규칙
- `docs/PROJECT_INDEX.md` - 프로젝트 구조 및 아키텍처
- `.github/workflows/on-pr-claude-review.yml` - 워크플로우 설정

### 유용한 명령어

```bash
# Claude Code 설정
claude setup-token                    # OAuth Token 생성
claude mcp list                       # MCP 서버 목록 확인
claude mcp add notion                 # Notion MCP 추가

# GitHub CLI
gh pr create                          # PR 생성
gh pr view                            # PR 확인
gh pr comment                         # PR 코멘트 추가

# GitHub Actions 로컬 테스트
act -l                                # 워크플로우 목록
act pull_request                      # PR 워크플로우 실행
```

---

## 📝 체크리스트 요약

### 초기 설정

- [ ] Claude Pro/Max 구독 확인
- [ ] 저장소 관리자 권한 확인
- [ ] Claude Code CLI 설치

### 인증 설정

- [ ] `claude setup-token` 실행 또는 `/install-github-app` 완료
- [ ] Claude GitHub 앱 설치
- [ ] `CLAUDE_CODE_OAUTH_TOKEN` Secret 추가

### 문서 작성

- [ ] `docs/PROJECT_INSIGHT.md` 작성
- [ ] `docs/PROJECT_INDEX.md` 작성
- [ ] `docs/PROJECT_INDEX.json` 작성 (선택)

### 워크플로우 설정

- [ ] `.github/workflows/on-pr-claude-review.yml` 생성
- [ ] 파일 경로 필터 커스터마이징
- [ ] 대상 브랜치 설정
- [ ] 리뷰 항목 조정

### Notion 연동 (선택)

- [ ] Notion Integration 생성
- [ ] Integration Token 복사
- [ ] `NOTION_API_KEY` Secret 추가
- [ ] Notion 페이지에 Integration 연결

### 테스트

- [ ] 테스트 PR 생성
- [ ] 워크플로우 실행 확인
- [ ] 리뷰 코멘트 확인
- [ ] Notion 연동 확인 (해당 시)

---

## 📌 문제 해결 연락처

- **Claude Code 문서**: https://code.claude.com/docs
- **GitHub Actions 문서**: https://docs.github.com/en/actions
- **Notion API 문서**: https://developers.notion.com

---

## 📝 변경 이력

### v1.1 (2025-11-11)
- ✅ `docs/CLAUDE.md` → `docs/PROJECT_INSIGHT.md` 파일명 통일
- ✅ `permissions: pull-requests: write` 수정 반영
- ✅ Notion MCP 조건부 설치 로직 추가
- ✅ 첫 워크플로우 추가 시 경고 해결 방법 추가
- ✅ 실제 워크플로우와 100% 일치하도록 업데이트

### v1.0 (2025-11-11)
- 초기 버전 작성

---

**작성자**: Claude Code
**마지막 업데이트**: 2025-11-11 v1.1
