# Blog Rules

## 포스트 작성
- 파일 위치: `_posts/`
- 파일명 형식: `YYYY-MM-DD-제목.md` (영문, 띄어쓰기 없이 하이픈으로)
- frontmatter 필수 항목: `title`, `date`, `categories`, `tags`
- 날짜 형식: `2026-05-26 00:00:00 +0900`

## Git
- 브랜치: `master`
- commitlint 규칙 적용됨 (husky)
- 커밋 형식: `type: 소문자 메시지`
- 포스트 추가/수정: `docs:` 사용
- 이미지 추가: `assets:` 사용

## 이미지
- 위치: `assets/img/posts/`
- 포스트에서 참조할 때 실제 파일 있는지 확인 후 작성

## 주의사항
- `.DS_Store` 절대 커밋 금지 (`.gitignore`에 등록됨)
- 이미지 경로 깨지면 GitHub Actions 빌드 실패함
