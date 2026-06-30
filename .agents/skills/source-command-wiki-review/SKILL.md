---
name: "source-command-wiki-review"
description: "위키 건강검진 — 고아·dead-link·모순·visibility·index 정합성 점검"
---

# source-command-wiki-review

Use this skill when the user asks to run the migrated source command `wiki-review`.

## Command Template

위키를 점검하고 정리 제안을 한다. 자동 수정하지 말고 제안 후 승인받는다.

1. **고아 노트**: 어떤 파일에서도 `[[링크]]`로 참조되지 않고, 스스로도 링크가 없는
   `notes/` 파일을 찾는다. (grep으로 `[[파일명]]` 역참조 확인)
2. **링크 누락**: source 노트인데 추출된 `notes/`로의 링크가 없는 경우.
3. **dead-link**: `[[링크]]` 대상이 실제 파일로 존재하지 않는 깨진 링크를 찾는다.
4. **모순 주장**: 서로 충돌하는 서술을 발견하면 `[!contradiction]` 콜아웃으로 표시 제안.
5. **visibility 누락**: frontmatter에 `visibility` 키가 없는 노트 목록.
   (없으면 private로 동작하지만 명시 권장)
6. **index 정합성**: `index.md`에 빠졌거나 사라진 페이지를 가리키는 항목을 찾는다.
7. **inbox 적체**: `inbox/`에 남은 파일 → 정리 대상.
8. **오래된 노트**: `updated` 가 오래된 핵심 노트(선택).

각 항목을 표로 보고하고, 어떤 링크/이동/정리를 할지 제안. 사용자 승인 후에만 변경.
