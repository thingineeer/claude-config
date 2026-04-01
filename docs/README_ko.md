🌐 [English](../README.md) | [한국어](README_ko.md) | [日本語](README_ja.md) | [中文](README_zh.md)

# claude-session-tools

> [Claude Code](https://claude.com/claude-code) 세션을 디바이스 간에 저장하고 복원합니다. 작업 컨텍스트를 다시는 잃어버리지 마세요.

이 플러그인이 도움이 되셨다면 [star](https://github.com/thingineeer/claude-session-tools)를 눌러주세요.

## 문제

Claude Code는 세션 히스토리를 `~/.claude/sessions/`에 로컬로 저장합니다. 이로 인해 두 가지 문제가 발생합니다:

1. **크로스 디바이스**: 컴퓨터를 바꾸면 컨텍스트가 사라집니다
2. **안정성**: 같은 기기에서도 `claude --continue`와 `--resume`은 로컬 세션 파일에 의존하기 때문에 업데이트 후 파일이 오래되거나, 손상되거나, 유실될 수 있습니다 — 결국 처음부터 다시 설명해야 합니다

## 해결 방법

**session-saver**는 로컬 세션 히스토리에 의존하지 않습니다. 전체 작업 컨텍스트를 **git에 커밋된 파일**로 저장하고, 자동으로 pull하고 복원하는 프로젝트 로컬 `/resume-{folder}` 커맨드를 생성합니다.

| 커맨드 | 범위 | 역할 |
|--------|------|------|
| `/session-saver:save-session` | Global (플러그인) | 컨텍스트 저장 + `/resume-{folder}` 생성 + push |
| `/resume-{folder}` | Project-local (자동 생성) | Auto-pull + 전체 컨텍스트 복원 |

> **참고**: 플러그인 스킬은 Claude Code 규칙에 따라 `/plugin-name:skill-name` 형식으로 네임스페이스됩니다. resume 커맨드는 프로젝트 로컬 스킬로 생성되므로 짧은 `/resume-{folder}` 형식을 사용합니다.

## 설치

[Claude Code](https://claude.com/claude-code) v1.0.33 이상이 필요합니다.

### 방법 1: 프롬프트

아래 프롬프트를 Claude Code에 붙여넣기하세요 — 기존 설정을 유지하면서 자동으로 플러그인을 설치합니다:

```
Add the following to ~/.claude/settings.json without removing any existing settings:

In enabledPlugins:
  "session-saver@claude-session-tools": true

In extraKnownMarketplaces:
  "claude-session-tools": { "source": { "source": "github", "repo": "thingineeer/claude-session-tools" } }
```

### 방법 2: 플러그인 메뉴

```
/plugins → Add marketplace → thingineeer/claude-session-tools → Install session-saver
```

설치 후 Claude Code를 재시작하면 끝입니다.

## 사용법

### 저장 (떠나기 전)

```
/session-saver:save-session
```

실행되는 작업:
1. 모든 미커밋 변경 사항을 논리적 단위로 나눠서 커밋
2. worktree 및 auto memory 정리
3. `docs/checkpoints/SESSION-STATE.md`에 전체 컨텍스트 저장
4. 프로젝트에 `.claude/skills/resume-{folder}/SKILL.md` 생성
5. 리모트에 push

### 복원 (어떤 기기에서든, 언제든)

```
/resume-{folder-name}
```

실행되는 작업:
1. `git fetch` — 리모트보다 뒤처져 있으면 자동 `git pull`
2. CLAUDE.md와 SESSION-STATE.md 읽기
3. 세이브 포인트에 기록된 key files 읽기
4. branch, 진행 상황, 다음 작업을 브리핑으로 출력

수동으로 `git pull`할 필요가 없습니다. resume 커맨드가 알아서 처리합니다.

## 지원 시나리오

| 시나리오 | 커맨드 |
|----------|--------|
| 기기 A → 기기 B (즉시) | B에서 `/resume-{folder}` — 자동 pull |
| 기기 A → 며칠 후 → 기기 B | 동일 — 세이브 포인트는 만료되지 않습니다 |
| 기기 A → 며칠 후 → 기기 A | 동일 — 최신 상태를 fetch해서 git에서 복원 |
| 기기 A → 앱 재시작 → 기기 A | 동일한 커맨드로 작동 |

## 동작 원리

```
기기 A                                      기기 B
  |                                           |
  |-- /session-saver:save-session             |
  |     |-- 변경 사항 커밋                      |
  |     |-- SESSION-STATE.md 작성              |
  |     |-- /resume-{folder} 스킬 생성         |
  |     |-- push                              |
  |                                           |
  |              git push ────────>           |
  |                                           |
  |                                           |-- /resume-{folder}
  |                                           |     |-- git fetch + auto pull
  |                                           |     |-- CLAUDE.md 읽기
  |                                           |     |-- SESSION-STATE.md 읽기
  |                                           |     |-- key files 읽기
  |                                           |     |-- 브리핑 출력
```

### 저장되는 것

| 파일 | 용도 |
|------|------|
| `docs/checkpoints/SESSION-STATE.md` | 세이브 포인트 — branch, 진행 상황, key files, 다음 작업 |
| `.claude/skills/resume-{folder}/SKILL.md` | 복원 스킬 — auto-pull + 컨텍스트 재구성 |

### 저장되지 않는 것

- 세션 대화 히스토리 (로컬에 유지)
- 시크릿, 인증 정보, `.env` 파일 (의도적으로 제외)
- 자동 생성 파일 (`node_modules/`, `Derived/`, `build/`)

## v1.0.x에서 마이그레이션

v1.0.x를 사용했다면 resume 커맨드가 `.claude/commands/resume-{folder}.md`로 생성되었습니다. v1.1.0부터는 `.claude/skills/resume-{folder}/SKILL.md`로 생성됩니다.

기존 스타일의 커맨드가 있는 프로젝트에서 `/session-saver:save-session`을 실행하면 자동으로 새 스킬 형식으로 마이그레이션하고 레거시 커맨드 파일을 삭제합니다.

## 커스터마이징

이 레포지토리를 fork하고 `plugins/session-saver/skills/save-session/SKILL.md`를 워크플로우에 맞게 수정하세요:

- SESSION-STATE.md 템플릿 변경
- 프로젝트별 빌드 검증 단계 추가
- 커밋 메시지 컨벤션 조정

## 기여

기여를 환영합니다! 이슈를 열거나 풀 리퀘스트를 제출해주세요.

1. 레포지토리 fork
2. feature branch 생성 (`git checkout -b feat/my-feature`)
3. [Conventional Commits](https://www.conventionalcommits.org/)에 따라 커밋
4. branch에 push하고 Pull Request 열기

## 라이선스

[MIT](../LICENSE)
