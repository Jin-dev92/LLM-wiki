# Serena MCP Setup Guide

> **Updated**: 2026-01-30
> **Purpose**: Team installation guide for Serena MCP (LSP-based code intelligence)

## What is Serena?

LSP(Language Server Protocol) 기반 코드 에이전트 툴킷. Claude Code에 IDE 수준의 코드 이해 능력을 추가한다.

**핵심 가치**: grep/glob 대신 심볼 단위 코드 탐색 + 편집. "이 함수를 누가 호출하는가?" 같은 질문에 정확히 답할 수 있다.

### 기존 Memory MCP와의 관계

| MCP | 역할 | 유지 여부 |
|-----|------|-----------|
| **Memory MCP** | 범용 지식 저장소 (벡터 검색, 태그, 세션 간 기억) | **유지** |
| **Serena** | 코드 심볼 탐색/편집 + 프로젝트 온보딩 | **추가** |

병행 사용한다. 역할이 다르다.

---

## Prerequisites

| 항목 | 요구사항 | 확인 명령 |
|------|----------|-----------|
| Python | 3.11+ | `python3 --version` |
| uv | latest | `uv --version` |
| uvx | latest (uv에 포함) | `uvx --version` |
| Git | any | `git --version` |

### uv 미설치 시

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Python 3.11+ 미설치 시

```bash
# uv로 설치 (권장)
uv python install 3.11

# 또는 brew
brew install python@3.11
```

> **Note**: macOS 기본 Python(3.9.6)으로는 부족하다. 3.11+ 필요.

---

## Installation

### Step 1: Claude Code에 Serena MCP 등록

**Global 등록 (모든 프로젝트에서 사용, 권장)**:

```bash
claude mcp add --scope user serena -- \
  uvx --from git+https://github.com/oraios/serena \
  serena start-mcp-server \
  --context=claude-code \
  --project-from-cwd
```

- `--scope user`: 전역 등록 (`~/.claude/settings.json`에 추가됨)
- `--context=claude-code`: Claude Code 전용 컨텍스트 (중복 도구 비활성화)
- `--project-from-cwd`: 현재 디렉토리 기준 프로젝트 자동 감지

**Per-project 등록 (특정 프로젝트만)**:

```bash
cd /path/to/project
claude mcp add serena -- \
  uvx --from git+https://github.com/oraios/serena \
  serena start-mcp-server \
  --context claude-code \
  --project "$(pwd)"
```

### Step 1-B: opencode에 Serena MCP 등록

`~/.config/opencode/opencode.json`의 `"mcp"` 블록에 추가:

```json
"serena": {
  "type": "local",
  "command": [
    "uvx",
    "--from",
    "git+https://github.com/oraios/serena",
    "serena",
    "start-mcp-server",
    "--context",
    "claude-code",
    "--project-from-cwd"
  ],
  "enabled": true
}
```

> opencode도 `claude-code` context를 사용한다. opencode 자체에는 전용 context가 없고, `claude-code`가 가장 적합하다.

### Step 2: 등록 확인

**Claude Code:**
```bash
claude mcp list
```

**opencode:** 실행 후 MCP 서버 목록에서 `serena` 확인.

`serena` 항목이 보이면 성공.

### Step 3: 프로젝트별 설정 (선택)

프로젝트 루트에 `.serena/project.yml` 생성:

```yaml
project_name: thub-fe-be
languages:
  - java
ignore_all_files_in_gitignore: true
ignored_paths:
  - "build/"
  - "target/"
  - ".gradle/"
  - "*.class"
  - "out/"
  - "node_modules/"
read_only: false
encoding: utf-8
```

Kotlin 프로젝트의 경우:

```yaml
project_name: thub-backend-service
languages:
  - kotlin
  - java
ignore_all_files_in_gitignore: true
ignored_paths:
  - "build/"
  - ".gradle/"
read_only: false
encoding: utf-8
```

Terraform 프로젝트의 경우:

```yaml
project_name: iac
languages:
  - terraform
ignore_all_files_in_gitignore: true
read_only: true
encoding: utf-8
```

> `.serena/` 디렉토리와 `project.yml`은 첫 활성화 시 자동 생성되기도 한다. 미리 만들어두면 온보딩이 더 빠르다.

### Step 4: 프로젝트 인덱싱 (대규모 프로젝트 권장)

```bash
cd /path/to/project
uvx --from git+https://github.com/oraios/serena serena project index
```

첫 심볼 조회 속도가 크게 개선된다. thub-fe-be처럼 큰 프로젝트에서 권장.

### Step 5: 온보딩

Claude Code 세션에서 첫 사용 시 Serena가 자동으로 온보딩을 수행한다.
- 프로젝트 구조 분석
- `.serena/memories/`에 프로젝트 컨텍스트 저장
- 이후 세션에서 저장된 메모리 활용

---

## Serena Tools (claude-code context)

`--context=claude-code` 사용 시 Claude Code 내장 기능과 중복되는 도구는 자동 비활성화된다. 주요 활성 도구:

### 심볼 탐색 (핵심)

| Tool | 용도 |
|------|------|
| `find_symbol` | 클래스/함수/변수 심볼 탐색 |
| `find_referencing_symbols` | 참조 추적 (IDE의 Find Usages) |
| `get_symbols_overview` | 파일의 심볼 구조 조회 |

### 심볼 편집

| Tool | 용도 |
|------|------|
| `replace_symbol_body` | 심볼 정의 전체 교체 |
| `insert_after_symbol` | 심볼 뒤에 코드 삽입 |
| `insert_before_symbol` | 심볼 앞에 코드 삽입 |
| `rename_symbol` | 리팩토링 (IDE의 Rename) |

### 메모리 (프로젝트 컨텍스트)

| Tool | 용도 |
|------|------|
| `write_memory` | 프로젝트 정보 저장 |
| `read_memory` | 저장된 메모리 조회 |
| `list_memories` | 메모리 목록 |
| `edit_memory` | 메모리 수정 |
| `delete_memory` | 메모리 삭제 |

### 워크플로우

| Tool | 용도 |
|------|------|
| `onboarding` | 프로젝트 온보딩 실행 |
| `check_onboarding_performed` | 온보딩 완료 여부 확인 |
| `initial_instructions` | 초기 지시 로드 |

---

## Troubleshooting

### Language Server 시작 실패

```
Error: Failed to start language server for java
```

**해결**: JDK가 PATH에 있는지 확인. Java LS는 초기 시작이 느리다 (특히 macOS). 첫 실행에서 수 분 걸릴 수 있다.

```bash
echo $JAVA_HOME
java --version
```

### Serena MCP 연결 실패

```bash
# MCP 서버 상태 확인
claude mcp list

# 재등록
claude mcp remove serena
claude mcp add --scope user serena -- \
  uvx --from git+https://github.com/oraios/serena \
  serena start-mcp-server \
  --context=claude-code \
  --project-from-cwd
```

### Python 버전 문제

Serena는 Python 3.11+ 필요. macOS 기본은 3.9.6.

```bash
# uv가 관리하는 Python 확인
uv python list

# 3.11 설치
uv python install 3.11
```

### 토큰 효율 최적화

Claude Code에서 Serena 도구가 많으면 도구 설명만으로 토큰을 소비한다. on-demand 로딩 활성화:

```bash
# 환경변수 설정 (필요 시)
ENABLE_TOOL_SEARCH=true
```

### Dashboard (디버깅용)

Serena 실행 중 `http://localhost:24282/dashboard/index.html` 에서 로그 확인 가능.

---

## .gitignore

프로젝트 `.gitignore`에 추가:

```
# Serena
.serena/memories/
```

> `project.yml`은 팀과 공유해도 된다. `memories/`는 개인 컨텍스트이므로 제외.

---

## References

- [Serena GitHub](https://github.com/oraios/serena)
- [Serena Documentation](https://oraios.github.io/serena)
- [Connecting Clients](https://oraios.github.io/serena/02-usage/030_clients.html)
- [Configuration](https://oraios.github.io/serena/02-usage/050_configuration.html)
