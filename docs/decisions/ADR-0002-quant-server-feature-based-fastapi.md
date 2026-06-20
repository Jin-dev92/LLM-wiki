# ADR-0002: quant-server FastAPI 아키텍처를 기능별(feature-based) 구조로 채택 (DDD 비채택)

## Status
Accepted

## Date
2026-06-20

## Context
신규 백엔드 `quant-server`(FastAPI, Python 3.13)의 코드 구조를 정한다. 막 스캐폴드된 상태
(placeholder 라우트뿐)이고, 앞으로 market data·strategy·order 등 **도메인이 여러 개로 갈라질 것**이
분명하다. 구조를 처음에 잡아두면 되돌리기 비용이 크므로 초기에 결정한다.

처음에는 DDD 4계층(domain/application/infrastructure/interfaces)으로 진행하려 했으나, 본인이
중단을 요청하고 "FastAPI에서 가장 자주 쓰는 구조"를 우선하기로 방향을 틀었다. 판단 기준:
**현 단계 규모 대비 과한 추상화를 피하고**(Karpathy: keep it simple), 기능 응집과 확장성은 확보.

## Decision
**기능별(feature-based) 구조를 채택한다.** `fastapi-best-practices`(zhanymkanov) 스타일로,
기능(도메인)별 폴더 안에서 router/schemas/service 정도만 얇게 분리한다.

```
src/quant_server/
  main.py        # create_app() + lifespan + 라우터 등록
  config.py      # pydantic-settings Settings
  health/        # 기능 슬라이스 예시
    router.py · schemas.py · service.py
```

- 새 기능 = `src/quant_server/<feature>/` 폴더 하나 추가(필요 시 models.py·dependencies.py).
- 의존 방향: router → service → (models/외부). 라우터에 비즈니스 로직 금지.
- 라우터 등록은 `main.py`의 `create_app()`에서만. 시작/종료 로직은 `lifespan`에.

함께 결정한 툴체인:
- 매니페스트 `pyproject.toml`(src 레이아웃, hatchling 빌드).
- **ruff**(린트+포맷), **mypy --strict**(타입), **pytest**(테스트). 설정은 모두 pyproject에.
- 패키지 관리는 **uv**(`uv sync` / `uv.lock` 커밋). 2026-06-20 도입 완료(brew 설치, editable에서 전환).

## Alternatives Considered

### A. DDD 4계층 (domain/application/infrastructure/interfaces)
- Pros: 도메인 로직 격리, 포트/어댑터로 인프라 교체 용이, 대규모 복잡 도메인에 강함.
- Cons: 현 스캐폴드 단계엔 계층·파일 수가 과함. Aggregate/VO/UoW까지 가면 무거움.
- Rejected: 현재 규모 대비 오버엔지니어링. 본인이 중단 요청.

### B. 계층별(타입별) 구조 — routers/ schemas/ models/ services/
- Pros: FastAPI 공식 "Bigger Applications" 예제 스타일, 소규모에 직관적.
- Cons: 기능이 늘면 한 기능 수정에 여러 폴더를 횡단해야 해 응집도 저하.
- Rejected: 도메인이 여러 개로 늘 게 확실한 quant-server엔 불리.

### C. 기능별 구조 (채택)
- Pros: 새 기능=폴더 하나, 관련 코드 한곳 응집, 마이크로서비스 분리 용이. de-facto 표준.
- Cons: 공통 로직 중복 가능 → 필요 시 `shared/`·`core/`로 추출하는 규율 필요.

## Consequences
- 실행 명령이 `uvicorn quant_server.main:app`으로 변경됨(PyCharm 실행 구성 갱신 필요).
- 검증 완료: ruff/mypy --strict 통과, `GET /health` → `200 {status: ok, version: 0.1.0}`.
- 루트의 옛 placeholder `main.py` 삭제 여부는 보류(확인 대기).
- **재검토 조건**: 단일 도메인의 비즈니스 규칙이 매우 복잡해지거나 인프라를 자주 교체해야 하면
  해당 기능 슬라이스에 한해 부분적으로 DDD 계층 도입을 검토하고 superseding ADR로 갱신.

## 근거 자료
- 프로젝트 컨벤션: `quant-server/CLAUDE.md` (아키텍처·코딩 규칙·명령어)
- 참고 스타일: fastapi-best-practices (zhanymkanov)
