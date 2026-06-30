---
name: "source-command-wiki-harvest"
description: "작업 중 발견한 재사용 가능한 지식과 규칙 후보를 llm-wiki로 수확한다"
---

# source-command-wiki-harvest

Use this skill when the user asks to run the migrated source command `wiki-harvest`.

## Command Template

현재 대화, 작업 로그, 변경 diff, 사용자가 제공한 자료에서 반복 재사용 가치가 있는 지식만 선별해
llm-wiki의 COMPILE 규칙에 맞춰 수확한다.

1. **후보 식별**: 일회성 실행 로그가 아니라 다음에도 쓸 수 있는 개념, 결정, 규칙, 워크플로우를 찾는다.
2. **계층 검색**: 먼저 `index.md`와 제목/태그/summary를 훑어 기존 노트와 중복 여부를 확인한다.
3. **분류**:
   - 개념 지식 -> `notes/`
   - 원천 대화/자료 요약 -> `sources/`
   - 운영 규칙 후보 -> `rules/`
   - 특정 작업 기록 -> `projects/`
4. **병합 우선**: 같은 개념 노트가 있으면 새 파일을 만들지 말고 기존 노트에 병합한다.
5. **민감정보 검사**: 키, 토큰, 고객정보, 비공개 링크가 있으면 저장하지 않고 사용자에게 확인한다.
6. **연결/카탈로그 갱신**: 관련 노트와 `[[링크]]`를 추가하고 `index.md`를 갱신한다.
7. **보고**: 생성/수정 파일, 병합한 노트, 저장하지 않은 민감 후보를 요약한다.

모르면 추측하지 말고 사용자에게 확인한다.
