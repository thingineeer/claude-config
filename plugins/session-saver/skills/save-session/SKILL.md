---
name: save-session
description: Save current session state and prepare to resume from any device. Use when ending a work session, switching computers, or before closing Claude Code.
disable-model-invocation: true
---

# Save Session

Save the current work context so it can be fully restored with `/resume-{folder}` — on any device, at any time.

## Core Principle

Claude Code session history (`~/.claude/sessions/`) is local and unreliable — it cannot transfer between devices, and even on the same machine `claude --continue` / `--resume` can fail when session files become stale or lost. This skill saves context as **git-committed files** that can reconstruct the full working state anywhere, regardless of local session state.

## Steps

### 1. Git status check

```bash
git status
git diff --stat
```

- Commit all unstaged changes, split by logical unit
- Restore auto-generated files (`Derived/`, `*.xcodeproj`, `node_modules/`, `build/`, etc.) — do not commit them
- Never commit files that may contain secrets (`.env`, `credentials.json`, `*.pem`, `*.key`)

### 2. Worktree cleanup

```bash
git worktree list
```

- Completed worktrees: merge/commit changes, then remove
- In-progress worktrees: record state in SESSION-STATE.md

### 3. Auto memory cleanup

Review the current project's auto memory (`~/.claude/projects/<project>/memory/`).

- Remove outdated or incorrect entries
- Save key discoveries from this session
- Keep `MEMORY.md` under 200 lines (split into topic files if needed)

### 4. Write SESSION-STATE.md

Create or update `docs/checkpoints/SESSION-STATE.md`. This is the **save point** — anyone reading this file should fully understand the current state.

```markdown
# Session State — {project name}

## Date
YYYY-MM-DD

## Branch
{branch name}

## Completed
- [x] Description of completed task
- [x] Description of completed task

## In Progress
- What was being worked on (be specific)
- Progress so far (file:line level detail)
- Immediate next step

## Remaining
- TODO item
- TODO item

## Key Files
Files to read first for full context:
- path/to/file — what it does
- path/to/file — what it does

## Notes
- Required environment variables or build config
- Known issues or blockers
- Recent dependency changes
```

### 5. Create resume skill (project-local)

Create or update `.claude/skills/resume-{folder}/SKILL.md` in the project, where `{folder}` is the current directory name.

This skill is **project-local** — it lives in the project repo and knows exactly which files to read. Users invoke it as `/resume-{folder}`.

The generated `SKILL.md` file must contain:

```markdown
---
name: resume-{folder}
description: Resume {project name} session — auto-pulls, reads save point, and prints briefing.
disable-model-invocation: true
---

# Resume — {project name}

## 1. Sync with remote
Run git fetch. If the local branch is behind, run git pull.
If diverged, warn the user — do NOT force pull.

## 2. Project rules
@CLAUDE.md

## 3. Work state
@docs/checkpoints/SESSION-STATE.md

## 4. Git status
Run git status and git branch, then show recent commits using the larger of:
- All commits from today: `git log --oneline --since="midnight"`
- Last 10 commits: `git log --oneline -10`
Use whichever returns more results.

## 5. Key files
Read all files listed in the "Key Files" section of SESSION-STATE.md.

## 6. Build environment
Verify build and test commands work.

## 7. Briefing
Print:

---
**Project**: {name}
**Branch**: {branch}
**Done**: {completed items}
**Current**: {in-progress work}
**Next**: {immediate next task}
---

Ask: "Ready to continue?"
```

### 6. Migrate legacy commands (if present)

If `.claude/commands/resume-{folder}.md` exists from a previous version, migrate it:

1. Create `.claude/skills/resume-{folder}/SKILL.md` with the content above
2. Delete `.claude/commands/resume-{folder}.md`
3. Commit the migration

### 7. Commit and push

Commit everything (SESSION-STATE.md + resume skill):

1. Stage changed files
2. Commit with conventional commit messages
3. Push to current branch

### 8. Final verification

```bash
git status   # must show: "nothing to commit, working tree clean"
```

Show recent commits using the larger of today's commits or last 10.

Print:

```
Session saved!

To restore (any device, any time):
  /resume-{folder}
```

## Supported Scenarios

| Scenario | How it works |
|----------|-------------|
| Device A → Device B (immediately) | `/resume-{folder}` on B — auto pulls |
| Device A → days later → Device B | Same — save point doesn't expire |
| Device A → days later → Device A | Same — fetches latest, restores from git |
| Device A → restart app → Device A | Same command works |

The save point is a git-committed file. It does not expire.

## Rules

- Working tree must be clean after save
- Push to current branch (main or feature)
- Never commit auto-generated files — restore them instead
- Never commit secrets (`.env`, credentials, keys)
- SESSION-STATE.md must be detailed enough for **anyone** to understand the full context
- The resume skill must include auto-pull so the user never has to `git pull` manually
- Flow: `/session-saver:save-session` → (any device, any time) → `/resume-{folder}`
