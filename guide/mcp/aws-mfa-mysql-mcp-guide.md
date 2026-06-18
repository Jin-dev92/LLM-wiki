# AWS MFA + MySQL MCP 설정 가이드

## 개요

Claude Code에서 MySQL MCP 서버를 사용하여 AWS RDS에 접근할 때, AWS MFA 인증을 자동으로 처리하는 방법을 안내합니다.

**핵심 구조**:
- `mfa` 프로필: 영구 키 (AKIA...) - MFA 갱신 시 사용
- `default` 프로필: 임시 키 (ASIA...) + 세션 토큰 - 실제 AWS API 호출 시 사용

---

## 0. 사전 준비 (본인 정보 확인)

아래 값들을 본인 환경에 맞게 확인 후 사용:

| 변수 | 설명 | 확인 방법 |
|------|------|-----------|
| `${AWS_ACCOUNT_ID}` | AWS 계정 ID | AWS Console 우측 상단 |
| `${MFA_DEVICE_NAME}` | MFA 디바이스명 | IAM > 본인 유저 > Security credentials > MFA |
| `${YOUR_PERMANENT_ACCESS_KEY}` | IAM 영구 Access Key | IAM > 본인 유저 > Security credentials |
| `${YOUR_PERMANENT_SECRET_KEY}` | IAM 영구 Secret Key | Access Key 생성 시 1회만 표시 |
| `${MYSQL_PASSWORD}` | MySQL 접속 비밀번호 | 팀 내부 공유 또는 Secrets Manager |

---

## 1. AWS Credentials 구조

### ~/.aws/credentials

```ini
[default]
# MFA 갱신 후 자동 설정되는 임시 키
aws_access_key_id = ASIAZ24ITBSE...      # 임시 (ASIA...)
aws_secret_access_key = h6ooB78SKCdR...
aws_session_token = FwoGZXIvYXdz...       # 12시간 후 만료

[mfa]
# 영구 키 (절대 변경 금지)
aws_access_key_id = ${YOUR_PERMANENT_ACCESS_KEY}      # 영구 (AKIA...)
aws_secret_access_key = ${YOUR_PERMANENT_SECRET_KEY}
```

**중요**:
- `mfa` 프로필의 영구 키는 MFA 갱신의 기반이므로 절대 변경하지 말 것
- `default` 프로필은 mfa-refresh 실행 시 자동으로 덮어씌워짐

---

## 2. mfa-refresh 함수 (zshrc)

### ~/.zshrc에 추가

```bash
mfa-refresh() {
  local mfa_serial="arn:aws:iam::${AWS_ACCOUNT_ID}:mfa/${MFA_DEVICE_NAME}"
  local source_profile="mfa"      # 영구 키 프로필
  local target_profile="default"  # 결과 저장 프로필

  if [ -z "$1" ]; then
    echo -n "MFA 코드: "
    read code
  else
    code=$1
  fi

  # 1. 만료된 세션 토큰 먼저 제거 (핵심!)
  aws configure set aws_session_token "" --profile "$target_profile"

  # 2. 영구 키(mfa 프로필)로 새 세션 토큰 발급
  creds=$(aws sts get-session-token \
    --serial-number "$mfa_serial" \
    --token-code "$code" \
    --profile "$source_profile" \
    --output json 2>&1)

  if echo "$creds" | grep -q "AccessKeyId"; then
    access_key=$(echo "$creds" | jq -r '.Credentials.AccessKeyId')
    secret_key=$(echo "$creds" | jq -r '.Credentials.SecretAccessKey')
    session_token=$(echo "$creds" | jq -r '.Credentials.SessionToken')
    expiration=$(echo "$creds" | jq -r '.Credentials.Expiration')

    # 3. default 프로필에 새 임시 키 저장
    aws configure set aws_access_key_id "$access_key" --profile "$target_profile"
    aws configure set aws_secret_access_key "$secret_key" --profile "$target_profile"
    aws configure set aws_session_token "$session_token" --profile "$target_profile"
    aws configure set region "ap-northeast-2" --profile "$target_profile"

    echo "MFA 세션 갱신 완료! (만료: $expiration)"
  else
    echo "MFA 세션 갱신 실패: $creds"
  fi
}
```

### 사용법

```bash
# 방법 1: 대화형
mfa-refresh
# MFA 코드: 123456

# 방법 2: 인자로 전달
mfa-refresh 123456
```

---

## 3. MySQL MCP 스크립트

### ~/.claude/scripts/mysql-mcp.sh

```bash
#!/bin/bash
# MySQL MCP Wrapper with MFA fallback
# Usage: mysql-mcp.sh <env>
# env: dev | prd

ENV=$1
AWS_PROFILE="default"

# ============================================
# 본인 환경에 맞게 수정
AWS_ACCOUNT_ID="${AWS_ACCOUNT_ID}"
MFA_DEVICE_NAME="${MFA_DEVICE_NAME}"
SECRET_PREFIX="thub"                    # Secrets Manager prefix
MYSQL_USER="claude_readonly"
MYSQL_PASSWORD="${MYSQL_PASSWORD}"
# ============================================

if [ -z "$ENV" ]; then
  echo "Usage: mysql-mcp.sh <dev|prd>" >&2
  exit 1
fi

get_db_host() {
  DB_URL=$(aws secretsmanager get-secret-value \
    --secret-id ${SECRET_PREFIX}/${ENV} \
    --profile ${AWS_PROFILE} \
    --query SecretString \
    --output text 2>/dev/null | jq -r '.db_url')

  if [ -n "$DB_URL" ] && [ "$DB_URL" != "null" ]; then
    echo "$DB_URL" | sed -n 's|jdbc:mysql://\([^:]*\):.*|\1|p'
    return 0
  fi
  return 1
}

refresh_mfa() {
  local mfa_serial="arn:aws:iam::${AWS_ACCOUNT_ID}:mfa/${MFA_DEVICE_NAME}"
  local source_profile="mfa"
  local target_profile="default"

  # macOS 다이얼로그로 MFA 입력 받기
  MFA_CODE=$(osascript -e 'display dialog "MFA 토큰 입력 (AWS Session 만료)" default answer "" buttons {"Cancel", "OK"} default button "OK"' -e 'text returned of result' 2>/dev/null)

  if [ -z "$MFA_CODE" ]; then
    osascript -e 'display alert "MFA 입력 취소됨" message "MySQL MCP 연결 실패"'
    exit 1
  fi

  # 핵심: 만료된 세션 토큰 먼저 제거
  aws configure set aws_session_token "" --profile "$target_profile"

  # 영구 키(mfa 프로필)로 새 세션 발급
  CREDS=$(aws sts get-session-token \
    --serial-number "$mfa_serial" \
    --token-code "$MFA_CODE" \
    --profile "$source_profile" \
    --output json 2>&1)

  if echo "$CREDS" | grep -q "AccessKeyId"; then
    aws configure set aws_access_key_id "$(echo $CREDS | jq -r '.Credentials.AccessKeyId')" --profile "$target_profile"
    aws configure set aws_secret_access_key "$(echo $CREDS | jq -r '.Credentials.SecretAccessKey')" --profile "$target_profile"
    aws configure set aws_session_token "$(echo $CREDS | jq -r '.Credentials.SessionToken')" --profile "$target_profile"
    aws configure set region "ap-northeast-2" --profile "$target_profile"
    return 0
  fi
  return 1
}

# 1차 시도
MYSQL_HOST=$(get_db_host)

# 실패 시 MFA 갱신 후 재시도
if [ -z "$MYSQL_HOST" ]; then
  refresh_mfa
  MYSQL_HOST=$(get_db_host)
fi

# 여전히 실패
if [ -z "$MYSQL_HOST" ]; then
  osascript -e 'display alert "MySQL MCP 연결 실패" message "AWS 인증 실패"'
  exit 1
fi

export MYSQL_HOST
export MYSQL_PORT=3306
export MYSQL_USER
export MYSQL_PASSWORD
export MYSQL_DATABASE="${SECRET_PREFIX}_db_${ENV}"

exec npx -y @liangshanli/mcp-server-mysql
```

### 스크립트 권한 설정

```bash
chmod 711 ~/.claude/scripts/mysql-mcp.sh
```

---

## 4. MCP 서버 등록

### ~/.claude.json 또는 프로젝트별 설정

```json
{
  "mcpServers": {
    "mysql-dev": {
      "type": "stdio",
      "command": "bash",
      "args": ["${HOME}/.claude/scripts/mysql-mcp.sh", "dev"]
    },
    "mysql-prd": {
      "type": "stdio",
      "command": "bash",
      "args": ["${HOME}/.claude/scripts/mysql-mcp.sh", "prd"]
    }
  }
}
```

### Claude CLI로 등록

```bash
# dev 환경
claude mcp add mysql-dev -- bash ~/.claude/scripts/mysql-mcp.sh dev

# prd 환경
claude mcp add mysql-prd -- bash ~/.claude/scripts/mysql-mcp.sh prd
```

---

## 5. 동작 흐름

```
[Claude Code 시작]
       |
       v
[mysql-mcp.sh 실행]
       |
       v
[Secrets Manager 접근 시도]
       |
   성공? ──Yes──> [MySQL MCP 서버 시작] ──> Connected!
       |
      No (세션 만료)
       |
       v
[MFA 다이얼로그 표시]
       |
       v
[세션 토큰 제거 + mfa 프로필로 갱신]
       |
       v
[Secrets Manager 재시도]
       |
       v
[MySQL MCP 서버 시작]
```

---

## 6. 문제 해결

### 6.1 ExpiredToken 오류

**증상**:
```
An error occurred (ExpiredToken) when calling the GetSessionToken operation:
The security token included in the request is expired
```

**원인**: 만료된 세션 토큰이 있는 상태에서 get-session-token 호출

**해결**:
```bash
# 세션 토큰 제거
aws configure set aws_session_token "" --profile default

# MFA 갱신
mfa-refresh
```

### 6.2 InvalidClientTokenId 오류

**증상**:
```
An error occurred (InvalidClientTokenId) when calling the GetCallerIdentity operation:
The security token included in the request is invalid
```

**원인**: default 프로필에 임시 키(ASIA...)만 있고 세션 토큰이 없음

**해결**:
```bash
# mfa 프로필의 영구 키를 default에 복원
aws configure set aws_access_key_id ${YOUR_PERMANENT_ACCESS_KEY} --profile default
aws configure set aws_secret_access_key ${YOUR_PERMANENT_SECRET_KEY} --profile default
aws configure set aws_session_token "" --profile default

# 그 다음 MFA 갱신
mfa-refresh
```

### 6.3 MCP 연결 실패 (mysql-dev: Failed to connect)

**체크리스트**:

1. AWS 인증 상태 확인:
   ```bash
   aws sts get-caller-identity --profile default
   ```

2. Secrets Manager 접근 확인:
   ```bash
   aws secretsmanager get-secret-value --secret-id thub/dev --query SecretString --output text | jq .db_url
   ```

3. 스크립트 직접 실행:
   ```bash
   bash -x ~/.claude/scripts/mysql-mcp.sh dev
   ```

4. MFA 프로필 영구 키 확인:
   ```bash
   cat ~/.aws/credentials | grep -A2 "\[mfa\]"
   # AKIA...로 시작해야 함
   ```

5. Claude Code 재시작:
   ```bash
   # 현재 세션 종료
   exit
   # 재시작
   claude
   ```

### 6.4 내일 아침 또 안 될 때 (자주 발생)

**원인**: 12시간 세션 만료 + default 프로필에 만료된 임시 키 잔존

**빠른 해결**:
```bash
# 1단계: 세션 토큰 제거
aws configure set aws_session_token "" --profile default

# 2단계: MFA 갱신
mfa-refresh
# (또는 Claude Code에서 MySQL MCP 사용 시 다이얼로그 자동 표시)

# 3단계: 확인
aws sts get-caller-identity
```

---

## 7. AWS Secrets Manager 구조

### Secret ID: ${SECRET_PREFIX}/dev, ${SECRET_PREFIX}/prd

```json
{
  "db_url": "jdbc:mysql://${MYSQL_HOST}:3306/${DATABASE_NAME}?...",
  "db_username": "...",
  "db_password": "..."
}
```

### MySQL 접속 정보

| 환경 | Host | Database | User |
|------|------|----------|------|
| dev | (Secrets Manager에서 조회) | ${SECRET_PREFIX}_db_dev | claude_readonly |
| prd | (Secrets Manager에서 조회) | ${SECRET_PREFIX}_db_prd | claude_readonly |

---

## 8. 직접 MySQL 접속 (MCP 우회)

MCP 없이 직접 접속이 필요한 경우:

```bash
# Secrets Manager에서 Host 조회
MYSQL_HOST=$(aws secretsmanager get-secret-value --secret-id thub/dev --query SecretString --output text | jq -r '.db_url' | sed -n 's|jdbc:mysql://\([^:]*\):.*|\1|p')

# 접속
mysql -h $MYSQL_HOST -u claude_readonly -p'${MYSQL_PASSWORD}' thub_db_dev

# 쿼리 실행
mysql -h $MYSQL_HOST -u claude_readonly -p'${MYSQL_PASSWORD}' thub_db_dev \
      -e "SELECT * FROM app_user LIMIT 10;"
```

---

## 9. 보안 권장사항

### 토큰/비밀번호 보안

- `~/.claude/scripts/mysql-mcp.sh`를 Git에 커밋하지 말 것
- AWS 영구 키(mfa 프로필)는 IAM에서 90일마다 로테이션 권장
- MySQL 비밀번호는 Secrets Manager로 이동 고려

### 권한 최소화

- `claude_readonly` 사용자는 SELECT 권한만 부여
- Secrets Manager 접근은 MFA 필수 정책 적용됨

---

## 10. 트러블슈팅 체크리스트

연결 안 될 때 순서대로 확인:

- [ ] `aws sts get-caller-identity` 성공하는가?
- [ ] 실패 시 `aws configure set aws_session_token "" --profile default` 실행
- [ ] `mfa-refresh` 실행 후 다시 확인
- [ ] `~/.aws/credentials`의 `[mfa]` 프로필에 영구 키(AKIA...) 있는가?
- [ ] `~/.claude/scripts/mysql-mcp.sh` 실행 권한(chmod 711) 있는가?
- [ ] `npx @liangshanli/mcp-server-mysql` 패키지 설치 가능한가?
- [ ] Claude Code 재시작 했는가?

---

## 11. 초기 설정 체크리스트

처음 설정 시 순서대로 진행:

- [ ] IAM에서 본인 Access Key 생성 (AKIA...)
- [ ] MFA 디바이스 등록 및 이름 확인
- [ ] `~/.aws/credentials`에 `[mfa]` 프로필 추가
- [ ] `~/.zshrc`에 `mfa-refresh` 함수 추가 후 `source ~/.zshrc`
- [ ] `~/.claude/scripts/` 디렉토리 생성
- [ ] `mysql-mcp.sh` 스크립트 생성 및 변수 수정
- [ ] `chmod 711 ~/.claude/scripts/mysql-mcp.sh`
- [ ] `claude mcp add mysql-dev ...` 등록
- [ ] `mfa-refresh` 실행하여 초기 세션 생성
- [ ] `claude mcp list`로 연결 확인

---

## 변경 이력

| 날짜 | 변경 내용 |
|------|-----------|
| 2025-12-20 | MCP 패키지 변경: `@benborla29/mcp-server-mysql` → `@liangshanli/mcp-server-mysql` (기존 패키지 의존성 버그) |
| 2025-12-19 | 최초 작성 |

---

## 작성 정보

- 최초 작성일: 2025-12-19
- 최종 수정일: 2025-12-20
- 카테고리: Infrastructure / AWS / MCP
- 관련 파일:
  - `~/.zshrc` (mfa-refresh 함수)
  - `~/.aws/credentials` (영구 키 + 임시 키)
  - `~/.claude/scripts/mysql-mcp.sh` (MCP 래퍼 스크립트)
