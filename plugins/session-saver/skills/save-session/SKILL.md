---
description: Save current session state and prepare to resume from any device. Use when ending a work session, switching computers, or before closing Claude Code.
disable-model-invocation: true
---

# /save-session — Save & Resume from Any Device

Save the current work context completely so you can resume from any device — even after restarting the app.

## Core Principle

Session history (`~/.claude/sessions/`) is local and cannot be transferred between devices.
Therefore, **context must be fully reconstructable from git-committed files**.

Three things needed for restoration:
1. `docs/checkpoints/SESSION-STATE.md` — Current work state snapshot
2. `.claude/commands/resume-{folder}.md` — Automated context restoration command
3. The code itself — Latest code pushed in a clean state

## Execution Steps

### 1. Check Git Status + Clean Staging

```bash
git status
git diff --stat
```

If there are unstaged changes, commit them all.
- Separate commits by logical units (no single bulk commit)
- Restore auto-generated files (Derived/, xcodeproj, node_modules, etc.)

### 2. Clean Up Worktrees

```bash
git worktree list
```

- Completed worktrees → commit and remove
- In-progress worktrees → record in SESSION-STATE.md

### 3. Clean Up Auto Memory

Check the current project's auto memory.

- Outdated or incorrect memories → delete/fix
- Key discoveries from this session → save to memory
- Keep MEMORY.md under 200 lines

### 4. Write SESSION-STATE.md (Critical)

Write `docs/checkpoints/SESSION-STATE.md` with current state.
This file is the **save point** — reading this alone should give full context on any device.

Required contents:

```markdown
# Session State — {project name}

## Date
{YYYY-MM-DD}

## Current Branch
{branch name}

## Completed Work
- [x] Task 1 description
- [x] Task 2 description

## In Progress
- What was being worked on (specific)
- How far along (file:line level)
- What to do immediately next

## Remaining Work
- TODO 1
- TODO 2

## Key Files (read these first to understand context)
- path/to/file1 — role description
- path/to/file2 — role description

## Notes / Things to Know
- Environment variables or settings needed for build
- Known issues
- Dependency changes
```

### 5. Write /resume-{folder} Command (Critical)

Write `.claude/commands/resume-{folder}.md` for the project.
When this command runs, context should be restored **as if picking up right where you left off**.

```markdown
# Session Restore — {project name}

Read the following files in order to understand current state.

## 1. Check Project Rules
@CLAUDE.md

## 2. Restore Work State
@docs/checkpoints/SESSION-STATE.md

## 3. Check Git Status
Run: git status, git log --oneline -5, git branch

## 4. Read Key Files
Read files listed in SESSION-STATE.md's "Key Files" section.

## 5. Check Build Environment
Verify build/test commands for the project

## 6. Output Briefing
Summarize current state in this format:

---
**Project**: {name}
**Branch**: {branch}
**Completed**: {summary of completed items}
**In Progress**: {what was being worked on}
**Next**: {task to start immediately}
---

Ask: "Ready to continue?"
```

### 6. Commit + Push

Commit all changes including SESSION-STATE.md and resume command.
Until staging is clean:
1. Add changed files
2. Commit (conventional commits)
3. Push

### 7. Final Check + Guidance

```bash
git status  # "nothing to commit, working tree clean"
git log --oneline -5
```

Output:

```
Session saved!

To continue on the same device:
  claude --continue

To continue on another device:
  git pull && claude
  → /resume-{folder}
```

## Rules
- Staging must be left in a clean state
- Push to the current branch (whether main or feature)
- Auto-generated files must not be committed (restore them)
- SESSION-STATE.md must be detailed enough for **anyone** to fully understand current state
- Resume command must **auto-read key files** to make context immediately available
- Save/Load flow: save-session → push → (other device) pull → /resume-{folder}
