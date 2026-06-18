# RDS Snapshot 로컬 복구 가이드

## 개요

AWS RDS 스냅샷을 S3 Parquet 형식으로 export한 후, 로컬 MySQL 데이터베이스로 복구하는 전체 절차를 설명합니다.

**작성일**: 2025-11-04
**대상 환경**: macOS, Docker MySQL 8, Python 3.9+

---

## 목차

1. [사전 준비사항](#1-사전-준비사항)
2. [S3에서 스냅샷 다운로드](#2-s3에서-스냅샷-다운로드)
3. [로컬 환경 구성](#3-로컬-환경-구성)
4. [복구 스크립트 실행](#4-복구-스크립트-실행)
5. [복구 결과 검증](#5-복구-결과-검증)
6. [트러블슈팅](#6-트러블슈팅)

---

## 1. 사전 준비사항

### 1.1 필수 도구

- **AWS CLI**: S3 접근을 위해 필요
- **Docker**: MySQL 컨테이너 실행
- **Python 3.9+**: Parquet 파일 변환용
- **충분한 디스크 공간**: 스냅샷 크기의 2배 이상 권장

### 1.2 AWS 설정

```bash
# AWS CLI 설치 확인
aws --version

# AWS 프로파일 설정 (기본 프로파일 사용)
aws configure --profile default

# S3 접근 권한 확인
aws s3 ls --profile default
```

### 1.3 Docker MySQL 컨테이너 준비

```bash
# MySQL 8 컨테이너 실행 (이미 실행 중이라면 스킵)
docker run --name some-mysql \
  -e MYSQL_ROOT_PASSWORD=my-secret-pw \
  -p 3306:3306 \
  -d mysql:8

# MySQL 컨테이너 상태 확인
docker ps | grep some-mysql
```

---

## 2. S3에서 스냅샷 다운로드

### 2.1 다운로드 디렉토리 생성

```bash
# 작업 디렉토리 생성
mkdir -p ~/snapshot
cd ~/snapshot
```

### 2.2 S3에서 스냅샷 다운로드

```bash
# RDS 스냅샷 export 데이터 다운로드
aws s3 cp \
  s3://cf-templates-pw7cuux92hoc-ap-northeast-2/rds-snapshot-export-s3-oct-20/ \
  ./snapshot \
  --recursive \
  --profile default
```

**다운로드 시간**: 데이터 크기에 따라 수십 분 ~ 수 시간 소요

### 2.3 다운로드 확인

```bash
# 다운로드된 파일 구조 확인
ls -lah snapshot/

# 예상 구조:
# snapshot/
# ├── thub_db_prd/
# │   ├── thub_db_prd.table1/
# │   │   └── 1/
# │   │       └── part-xxxxx.gz.parquet
# │   ├── thub_db_prd.table2/
# │   └── ...
# ├── export_info_*.json
# └── export_tables_info_*.json
```

---

## 3. 로컬 환경 구성

### 3.1 Python 패키지 설치

```bash
# 필수 Python 라이브러리 설치
pip3 install pandas pyarrow mysql-connector-python
```

설치되는 패키지:
- `pandas`: 데이터 처리
- `pyarrow`: Parquet 파일 읽기
- `mysql-connector-python`: MySQL 연결

### 3.2 MySQL 데이터베이스 생성

```bash
# 데이터베이스 생성
docker exec some-mysql mysql -u root -pmy-secret-pw \
  -e "CREATE DATABASE IF NOT EXISTS thub_db_prd CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 생성 확인
docker exec some-mysql mysql -u root -pmy-secret-pw \
  -e "SHOW DATABASES LIKE 'thub_db_prd';"
```

---

## 4. 복구 스크립트 실행

### 4.1 복구 스크립트 다운로드/생성

복구 스크립트 `restore_auto.py`를 작업 디렉토리에 준비합니다.

**스크립트 위치**: `/Users/bj/snapshot/restore_auto.py`

### 4.2 스크립트 설정 확인

스크립트 내 MySQL 연결 정보를 확인/수정합니다:

```python
db_config = {
    "host": "localhost",
    "user": "root",
    "password": "my-secret-pw",
    "database": "thub_db_prd",
    "port": 3306
}
```

### 4.3 복구 실행

```bash
cd ~/snapshot
python3 restore_auto.py
```

### 4.4 복구 프로세스

스크립트는 다음 단계를 자동으로 수행합니다:

1. **MySQL 연결**: 로컬 MySQL 데이터베이스 연결
2. **테이블 목록 스캔**: Parquet 디렉토리에서 테이블 목록 추출 (352개)
3. **각 테이블별 처리**:
   - Parquet 파일 읽기
   - DataFrame으로 변환
   - MySQL 테이블 생성 (자동 스키마 추론)
   - 데이터 배치 삽입 (1,000 rows/batch)
4. **로그 기록**: 모든 작업은 `restore_log.txt`에 기록

### 4.5 실시간 모니터링

복구 진행 상황을 실시간으로 확인:

```bash
# 실시간 로그 보기
tail -f ~/snapshot/restore_log.txt

# 진행률 확인
grep "진행:" ~/snapshot/restore_log.txt | tail -1

# 에러 확인
grep "에러:" ~/snapshot/restore_log.txt
```

**예상 소요 시간**: 약 2-5분 (데이터 크기에 따라 변동)

---

## 5. 복구 결과 검증

### 5.1 복구 통계 확인

```bash
# 최종 결과 확인
grep -E "(성공:|실패:)" ~/snapshot/restore_log.txt | tail -5
```

**예상 출력**:
```
[2025-11-04 14:01:14]   성공: 222/352
[2025-11-04 14:01:14]   실패: 72
```

### 5.2 데이터베이스 검증

```bash
# 복구된 테이블 개수 확인
docker exec some-mysql mysql -u root -pmy-secret-pw thub_db_prd \
  -e "SELECT COUNT(*) as table_count FROM information_schema.tables WHERE table_schema = 'thub_db_prd';"

# 주요 테이블 데이터 확인
docker exec some-mysql mysql -u root -pmy-secret-pw thub_db_prd \
  -e "SELECT table_name, table_rows FROM information_schema.tables WHERE table_schema = 'thub_db_prd' ORDER BY table_rows DESC LIMIT 10;"
```

**예상 결과**:
```
table_count
294

TABLE_NAME                                  TABLE_ROWS
top_exposure_crawl_data                     5,690,821
vegas_parse_log_240707                      1,030,052
ttc_clinic_schedule_info_change_record        258,250
crm_new_daily_record                          130,699
keyword_alarm                                  73,530
```

### 5.3 샘플 데이터 조회

```bash
# 특정 테이블 데이터 확인
docker exec some-mysql mysql -u root -pmy-secret-pw thub_db_prd \
  -e "SELECT COUNT(*) FROM customer_info;"

docker exec some-mysql mysql -u root -pmy-secret-pw thub_db_prd \
  -e "SELECT * FROM customer_info LIMIT 5;"
```

---

## 6. 트러블슈팅

### 6.1 알려진 이슈

#### 이슈 1: NaN 값 처리 오류

**에러 메시지**:
```
1054 (42S22): Unknown column 'nan' in 'field list'
```

**원인**: Pandas DataFrame에서 컬럼명이 NaN인 경우 발생

**해결방법**: 해당 테이블은 스킵되며, 필요시 수동 복구 필요

#### 이슈 2: Numpy 타입 변환 오류

**에러 메시지**:
```
Failed executing the operation; Python type numpy.int64 cannot be converted
```

**원인**: Numpy 데이터 타입을 MySQL이 직접 처리하지 못함

**해결방법**: 스크립트에서 `.astype(int)` 등으로 타입 변환 필요

### 6.2 실패한 테이블 재처리

총 72개 테이블이 실패한 경우, 다음 카테고리로 분류됩니다:

1. **데이터 없음** (58개): 빈 테이블, 복구 불필요
2. **NaN 컬럼** (약 50개): 컬럼명 문제
3. **타입 변환 실패** (약 10개): Numpy 타입 이슈

필요한 테이블만 선택적으로 수동 복구 권장

### 6.3 MySQL 연결 오류

**에러**: Connection refused

**확인사항**:
1. Docker 컨테이너 실행 상태
2. MySQL 포트 (3306) 충돌 여부
3. 방화벽 설정

```bash
# Docker 컨테이너 상태 확인
docker ps -a | grep some-mysql

# 컨테이너 재시작
docker restart some-mysql

# 포트 확인
netstat -an | grep 3306
```

### 6.4 디스크 공간 부족

**증상**: 복구 중 중단

**해결**:
```bash
# 디스크 사용량 확인
df -h

# 불필요한 파일 삭제
rm -rf ~/snapshot/snapshot  # 복구 완료 후 삭제 가능
```

---

## 복구 완료 후 정리

### 선택적 정리 작업

```bash
# 다운로드한 스냅샷 파일 삭제 (용량 확보)
rm -rf ~/snapshot/snapshot

# 로그 파일 백업
cp ~/snapshot/restore_log.txt ~/snapshot/restore_log_$(date +%Y%m%d).txt
```

### 복구 스크립트 보관

향후 재사용을 위해 스크립트를 버전 관리:

```bash
# Git 저장소에 커밋 (선택사항)
cd ~/snapshot
git add restore_auto.py RDS_SNAPSHOT_RESTORE_GUIDE.md
git commit -m "Add RDS snapshot restore scripts and documentation"
```

---

## 부록

### A. 복구 스크립트 주요 기능

1. **자동 스키마 추론**: Parquet 메타데이터에서 MySQL 스키마 생성
2. **배치 처리**: 메모리 효율적인 1,000 rows 단위 삽입
3. **에러 핸들링**: 개별 테이블 실패 시 다음 테이블 계속 진행
4. **진행률 표시**: 실시간 진행 상황 로그
5. **자동 로그 기록**: 전체 복구 과정 파일 저장

### B. 성능 최적화 팁

1. **대용량 테이블**: `batch_size` 조정 (기본 1,000)
2. **병렬 처리**: 멀티프로세싱 적용 가능
3. **인덱스**: 복구 후 필요한 인덱스 수동 생성

### C. 데이터 타입 매핑

| Parquet/Pandas 타입 | MySQL 타입 |
|-------------------|-----------|
| int64, int32      | BIGINT    |
| float64, double   | DOUBLE    |
| bool              | BOOLEAN   |
| datetime64        | DATETIME  |
| object, string    | TEXT      |

---

## 문의 및 지원

복구 중 문제 발생 시:
1. `restore_log.txt` 로그 확인
2. 에러 메시지 검색
3. 스크립트 수정 또는 수동 복구 진행

**작성자**: Claude
**최종 수정**: 2025-11-04
