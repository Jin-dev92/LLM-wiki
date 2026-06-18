---
description: 소스(유튜브/웹/로컬파일/메모)를 읽어 위키화한다
argument-hint: "<url | 파일경로 | 유튜브링크>  (없으면 대화/메모 입력)"
---

입력: $ARGUMENTS

CLAUDE.md의 COMPILE 단계를 그대로 수행한다.

1. **소스 판별 & 읽기**
   - youtube.com / youtu.be → `watch` 스킬로 내용 추출 (ingested_via: youtube)
   - http(s) URL → WebFetch (ingested_via: web)
   - 로컬 경로(.pdf/.md/.txt/이미지) → Read (ingested_via: pdf)
   - 인자 없음/붙여넣은 텍스트 → 대화 내용을 소스로 (ingested_via: memo)
2. **연관성 검색(계층)**: 먼저 `index.md`와 노트 제목/태그/summary를 훑어 관련 후보를
   좁힌다. 필요한 노트만 본문을 연다.
3. **원천 기록 생성**: `templates/source.md` 기반으로 `sources/<제목-kebab>.md` 작성.
   summary + url/author/ingested_via + created/updated(오늘) 채움.
4. **영구노트 추출/병합**: 핵심 개념을 `templates/note.md` 기반으로 `notes/<개념-kebab>.md`로.
   **원문 복붙 금지, 내 언어로 재구성.** 개념이 여러 개면 파일도 여러 개(atomic).
   **이미 같은 개념 노트가 있으면 새로 만들지 말고 병합**, 출처를 누적. summary 채우고,
   추론한 문장은 `provenance: inferred`(기본 extracted)로 표시.
5. **양방향 링크**: source ↔ note, note ↔ 기존 노트를 `[[링크]]`로 연결.
   관련 `MOC/` 파일이 있으면 항목 추가, 없고 주제가 크면 새 MOC 제안.
6. **카탈로그 갱신**: 생성/병합한 페이지를 `index.md`에 추가/수정.
7. **보고**: 생성/수정한 파일 목록 + 연결한 링크 + 검토 필요 항목을 요약 출력.

기본 visibility는 private. 사용자가 공유용이라 명시하면 team/public.
