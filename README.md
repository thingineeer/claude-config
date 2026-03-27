# session-saver

> Save and restore Claude Code sessions across devices. Never lose your work context again.

## Problem

Claude Code session history is stored locally (`~/.claude/sessions/`). When you switch to another computer or restart the app, your context is gone.

## Solution

`/session-saver:save-session` saves your entire work context as git-committed files:

- **SESSION-STATE.md** — A snapshot of what you were doing, key files, and next steps
- **/resume-{folder}** — A slash command that auto-restores context on any device

## Install

```bash
/plugin marketplace add thingineeer/claude-config
/plugin install session-saver@claude-session-tools
```

## Usage

### Save (before leaving)

```
/session-saver:save-session
```

This will:
1. Commit all changes (split by logical units)
2. Clean up worktrees and auto memory
3. Write `SESSION-STATE.md` with full context
4. Create `/resume-{folder}` command for the project
5. Push everything

### Resume (on another device)

```bash
git pull && claude
```

Then run:

```
/resume-{folder-name}
```

Context is fully restored — branch, progress, key files, next steps.

## How It Works

```
[Device A]                          [Device B]
    │                                   │
    ├─ /session-saver:save-session      │
    │   ├─ commit changes               │
    │   ├─ write SESSION-STATE.md       │
    │   ├─ create /resume-{folder}      │
    │   └─ push                         │
    │                                   │
    │              git push ──────►     │
    │                                   ├─ git pull
    │                                   ├─ claude
    │                                   └─ /resume-{folder}
    │                                       ├─ read CLAUDE.md
    │                                       ├─ read SESSION-STATE.md
    │                                       ├─ read key files
    │                                       └─ briefing output
```

## License

MIT
