# CLAUDE.md

## 브랜치 전략

이 프로젝트는 오픈소스이므로 브랜치 기반으로 개발합니다.

### 규칙

- **main 브랜치에 직접 커밋 금지** (README 수정 제외)
- feature/fix 작업은 반드시 브랜치를 생성하고 PR을 통해 머지
- 브랜치 네이밍: `feat/{feature-name}`, `fix/{bug-name}`, `chore/{task-name}`, `docs/{doc-name}`
- PR 머지 시 squash 금지, 일반 머지 사용

### 워크플로우

1. `main`에서 브랜치 생성
2. 작업 후 커밋 (conventional commits)
3. push 후 PR 생성
4. 리뷰 후 머지
