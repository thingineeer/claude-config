# /save-session — 세션 저장 + 어디서든 이어서 작업

현재 작업 컨텍스트를 완전히 저장합니다.
앱 재부팅, 다른 컴퓨터 — 어디서든 `/resume-{폴더명}`만 실행하면 방금 하던 작업처럼 이어서 할 수 있어야 합니다.

## 핵심 원칙

세션 기록(`~/.claude/sessions/`)은 로컬이라 다른 컴퓨터로 옮길 수 없습니다.
따라서 **git에 커밋되는 파일들로 컨텍스트를 완전히 재구성**할 수 있어야 합니다.

복원에 필요한 3가지:
1. `docs/checkpoints/SESSION-STATE.md` — 현재 작업 상태 스냅샷
2. `.claude/commands/resume-{폴더명}.md` — 컨텍스트 복원 자동화 커맨드
3. 코드 자체 — clean 상태로 push된 최신 코드

## 실행 순서

### 1. Git 상태 확인 + staging 정리

```bash
git status
git diff --stat
```

unstaged 변경사항이 있으면 모두 커밋합니다.
- 기능 단위로 분리 커밋 (한 덩어리 금지)
- Derived/, xcodeproj 등 자동 생성 파일은 restore

### 2. Worktree 정리

```bash
git worktree list
```

- 완료된 worktree → 커밋 후 제거
- 진행 중인 worktree → SESSION-STATE.md에 기록

### 3. 자동 메모리 정리

현재 프로젝트의 자동 메모리를 확인합니다.

- 오래되거나 틀린 메모리 → 삭제/수정
- 이번 세션의 핵심 발견사항 → 메모리에 저장
- MEMORY.md 200줄 이하 유지

### 4. SESSION-STATE.md 작성 (핵심)

`docs/checkpoints/SESSION-STATE.md`를 현재 상태로 작성합니다.
이 파일이 **세이브 포인트**이므로, 다른 컴퓨터에서 이것만 읽으면 전체 맥락을 파악할 수 있어야 합니다.

필수 포함 내용:

```markdown
# Session State — {프로젝트명}

## 날짜
{YYYY-MM-DD}

## 현재 브랜치
{브랜치명}

## 완료된 작업
- [ ] 작업1 설명
- [ ] 작업2 설명

## 진행 중인 작업
- 현재 하고 있던 것 (구체적으로)
- 어디까지 했는지 (파일명:라인 수준)
- 다음에 바로 해야 할 것

## 남은 작업
- TODO 1
- TODO 2

## 핵심 파일 (이 파일들을 먼저 읽으면 컨텍스트 파악 가능)
- path/to/file1.swift — 역할 설명
- path/to/file2.ts — 역할 설명

## 주의사항 / 알아둘 것
- 빌드 시 필요한 환경변수나 설정
- 알려진 이슈
- 의존성 변경사항
```

### 5. /resume-{폴더명} 커맨드 작성 (핵심)

프로젝트의 `.claude/commands/resume-{폴더명}.md`를 작성합니다.
이 커맨드가 실행되면 **방금 하던 작업처럼** 컨텍스트가 복원되어야 합니다.

```markdown
# 세션 복원 — {프로젝트명}

다음 파일들을 순서대로 읽고 현재 상태를 파악합니다.

## 1. 프로젝트 규칙 확인
@CLAUDE.md

## 2. 현재 작업 상태 복원
@docs/checkpoints/SESSION-STATE.md

## 3. Git 상태 확인
git status, git log --oneline -5, git branch 실행

## 4. 핵심 파일 읽기
SESSION-STATE.md의 "핵심 파일" 섹션에 나열된 파일들을 읽습니다.

## 5. 빌드 환경 확인
프로젝트에 맞는 빌드/테스트 명령 확인

## 6. 브리핑 출력
다음 형식으로 현재 상태를 요약합니다:

---
**프로젝트**: {이름}
**브랜치**: {브랜치}
**완료**: {완료 항목 요약}
**진행 중**: {현재 하던 것}
**다음 할 일**: {바로 시작할 작업}
---

"이어서 진행할까요?" 라고 물어봅니다.
```

### 6. 커밋 + Push

SESSION-STATE.md와 resume 커맨드 포함하여 모든 변경사항을 커밋합니다.
staging이 깨끗해질 때까지:
1. 변경된 파일 add
2. 커밋 (한글 conventional commits)
3. push

### 7. 최종 확인 + 안내

```bash
git status  # "nothing to commit, working tree clean"
git log --oneline -5
```

마지막으로 출력:

```
세션 저장 완료!

이어서 작업하려면 (같은 컴퓨터):
  claude --continue

이어서 작업하려면 (다른 컴퓨터):
  git pull && claude
  → /resume-{폴더명}
```

## 행동 규칙
- staging은 반드시 clean 상태로 만들 것
- push는 현재 브랜치에 (main이든 feature든)
- 커밋 메시지는 한글 (conventional commits)
- Derived/, *.xcodeproj 등 자동 생성 파일은 커밋하지 말 것 (restore)
- SESSION-STATE.md는 **다른 사람이 읽어도** 현재 상태를 완전히 파악할 수 있을 만큼 상세하게
- resume 커맨드는 **핵심 파일까지 자동으로 읽어서** 바로 작업 가능한 상태로 만들 것
- 세이브/로드 흐름: save-session → push → (다른 컴퓨터) pull → /resume-{폴더명}
