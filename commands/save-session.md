# /save-session — 세션 저장 + 다른 컴퓨터에서 이어서 작업

현재 작업 상태를 깔끔하게 저장하고, 다른 컴퓨터에서 pull 받아서 이어서 작업할 수 있게 준비합니다.

## 실행 순서

### 1. Git 상태 확인 + staging 정리

```bash
git status
git diff --stat
```

unstaged 변경사항이 있으면 모두 커밋합니다.
- 기능 단위로 분리 커밋 (한 덩어리 금지)
- Derived/, xcodeproj 등 자동 생성 파일은 restore

### 2. SESSION-STATE.md 업데이트

프로젝트 루트에 `docs/checkpoints/SESSION-STATE.md`가 있으면 현재 상태로 업데이트:
- 현재 브랜치
- 완료된 작업
- 진행 중인 작업
- 남은 작업
- 핵심 파일 목록

없으면 생성합니다.

### 3. /resume-{폴더명} 슬래시 커맨드 업데이트

프로젝트의 `.claude/commands/resume-{폴더명}.md`가 있으면 최신 상태로 업데이트.
없으면 새로 생성:

```
프로젝트 .claude/commands/resume-{현재폴더명}.md
```

내용:
- 메모리 로드 경로
- CLAUDE.md 읽기
- SESSION-STATE.md 읽기
- Git 상태 확인
- 빌드 환경 확인
- 브리핑 출력 (현재 버전, 완료/진행 중/남은 작업)

### 4. 커밋 + Push

staging이 깨끗해질 때까지:
1. 변경된 파일 add
2. 커밋 (한글 conventional commits)
3. push

### 5. 최종 확인

```bash
git status  # "nothing to commit, working tree clean" 확인
git log --oneline -5  # 최근 커밋 확인
```

## 행동 규칙
- staging은 반드시 clean 상태로 만들 것
- push는 현재 브랜치에 (main이든 feature든)
- 커밋 메시지는 한글 (conventional commits)
- Derived/, *.xcodeproj 등 자동 생성 파일은 커밋하지 말 것 (restore)
- 다른 컴퓨터에서 `/resume-{폴더명}`으로 바로 이어서 할 수 있어야 함
