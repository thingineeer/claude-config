# claude-session-tools

> Save and restore [Claude Code](https://claude.com/claude-code) sessions across devices. Never lose your work context again.

If this plugin saved your time, consider giving it a [star](https://github.com/thingineeer/claude-session-tools).

## The Problem

Claude Code stores session history locally in `~/.claude/sessions/`. This creates two problems:

1. **Cross-device**: When you switch computers, your context is gone
2. **Reliability**: Even on the same device, `claude --continue` and `--resume` depend on local session files that can become stale, corrupted, or lost after updates — leaving you to re-explain everything from scratch

## The Solution

**session-saver** does not depend on local session history. It saves your entire work context as **git-committed files** and generates a project-local `/resume-{folder}` command that auto-pulls and restores everything.

| Command | Scope | What it does |
|---------|-------|-------------|
| `/session-saver:save-session` | Global (plugin) | Save context + generate `/resume-{folder}` + push |
| `/resume-{folder}` | Project-local (auto-generated) | Auto-pull + restore full context |

## Installation

Requires [Claude Code](https://claude.com/claude-code) v1.0.33 or later.

```
/plugin marketplace add thingineeer/claude-session-tools
/plugin install session-saver@claude-session-tools
```

## Usage

### Save (before leaving)

```
/session-saver:save-session
```

This will:
1. Commit all pending changes (split by logical unit)
2. Clean up worktrees and auto memory
3. Create `docs/checkpoints/SESSION-STATE.md` with full context
4. Generate `.claude/commands/resume-{folder}.md` for the project
5. Push everything to remote

### Restore (any device, any time)

```
/resume-{folder-name}
```

This will:
1. `git fetch` — if behind remote, auto `git pull`
2. Read CLAUDE.md and SESSION-STATE.md
3. Read key files listed in the save point
4. Print a briefing with branch, progress, and next steps

No need to manually `git pull`. The resume command handles it.

## Supported Scenarios

| Scenario | Command |
|----------|---------|
| Device A → Device B (immediately) | `/resume-{folder}` on B — auto pulls |
| Device A → days later → Device B | Same — save point doesn't expire |
| Device A → days later → Device A | Same — fetches latest, restores from git |
| Device A → restart app → Device A | Same command works |

## How It Works

```
Device A                              Device B
  |                                     |
  |-- /session-saver:save-session       |
  |     |-- commit changes              |
  |     |-- write SESSION-STATE.md      |
  |     |-- generate /resume-{folder}   |
  |     |-- push                        |
  |                                     |
  |            git push ────────>       |
  |                                     |
  |                                     |-- /resume-{folder}
  |                                     |     |-- git fetch + auto pull
  |                                     |     |-- read CLAUDE.md
  |                                     |     |-- read SESSION-STATE.md
  |                                     |     |-- read key files
  |                                     |     |-- print briefing
```

### What gets saved

| File | Purpose |
|------|---------|
| `docs/checkpoints/SESSION-STATE.md` | Save point — branch, progress, key files, next steps |
| `.claude/commands/resume-{folder}.md` | Restore command — auto-pull + context rebuild |

### What does NOT get saved

- Session conversation history (stays local)
- Secrets, credentials, `.env` files (excluded by design)
- Auto-generated files (`node_modules/`, `Derived/`, `build/`)

## Customization

Fork this repository and modify `plugins/session-saver/skills/save-session/SKILL.md` to fit your workflow:

- Change the SESSION-STATE.md template
- Add project-specific build verification steps
- Adjust commit message conventions

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

1. Fork the repository
2. Create a feature branch (`git checkout -b feat/my-feature`)
3. Commit your changes using [Conventional Commits](https://www.conventionalcommits.org/)
4. Push to the branch and open a Pull Request

## License

[MIT](LICENSE)
