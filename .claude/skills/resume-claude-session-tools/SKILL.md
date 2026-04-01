---
name: resume-claude-session-tools
description: Resume claude-session-tools session — auto-pulls, reads save point, and prints briefing.
disable-model-invocation: true
---

# Resume — claude-session-tools

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
**Project**: claude-session-tools
**Branch**: {branch}
**Done**: {completed items}
**Current**: {in-progress work}
**Next**: {immediate next task}
---

Ask: "Ready to continue?"
