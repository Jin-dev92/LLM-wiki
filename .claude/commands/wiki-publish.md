---
description: visibility team|public 인 노트만 대상 위치로 export (유출 검사 포함)
argument-hint: "<export 대상 위치(폴더/레포 경로)>"
---

대상 위치: $ARGUMENTS

1. **공유 후보 수집**: frontmatter `visibility: team` 또는 `public` 인 파일만 수집.
   `visibility` 없거나 `private` 인 파일은 제외(안전 기본값).
2. **유출 방지 검사 (핵심)**: 수집한 각 파일의 본문 `[[링크]]` 대상을 확인.
   대상이 `private`(또는 미수집) 노트면 → **경고하고 export 중단.**
   깨진 링크/정보 유출 위험을 사람이 해소하도록 위반 목록을 보고.
3. **export**: 검사를 통과하면 수집 파일을 대상 위치로 복사(폴더 구조 유지).
4. **보고**: export한 파일 수/목록, 제외된 private 수, 발견된 위반을 요약.

동기화(git)와 공유(export)는 분리된 행위임을 명심 — 이 커맨드만 외부로 내보낸다.
