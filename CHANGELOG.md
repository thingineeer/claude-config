# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-03-31

### Breaking Changes

- Resume command now generated as `.claude/skills/resume-{folder}/SKILL.md` instead of `.claude/commands/resume-{folder}.md`
- Save command is now invoked as `/session-saver:save-session` (plugin namespace)

### Fixed

- `/save-session` did not work — plugin commands require namespace prefix (`/session-saver:save-session`). Removed misleading `commands/` directory that implied `/save-session` would work directly
- Resume command generated as `.claude/commands/` was unreliable — skills take precedence over commands and provide frontmatter support (`name`, `description`, `disable-model-invocation`)

### Added

- `name` field in SKILL.md frontmatter for explicit skill registration
- `@CLAUDE.md` and `@docs/checkpoints/SESSION-STATE.md` references in resume template for automatic file loading
- Legacy command migration step: auto-converts old `.claude/commands/resume-{folder}.md` to new skill format

### Changed

- README updated to reflect correct `/session-saver:save-session` invocation
- Diagram and file references updated to match new skill structure

## [1.0.1] - 2025-03-27

### Added

- `/save-session` shortcut via `commands/` directory (no namespace prefix needed)
- Smart commit log range: uses larger of today's commits or last 10

### Changed

- Updated README to reflect `/save-session` usage

## [1.0.0] - 2025-03-27

### Added

- `session-saver` plugin with `save-session` skill
- SESSION-STATE.md save point template for cross-device context restoration
- Auto-generated `/resume-{folder}` project-local command with git auto-pull
- Worktree cleanup step
- Auto memory cleanup step
- Security rules: never commit `.env`, credentials, or secret files
- Support for 4 scenarios: cross-device, delayed resume, same-device restart
- Claude Code plugin marketplace support
- MIT license

[1.1.0]: https://github.com/thingineeer/claude-session-tools/releases/tag/v1.1.0
[1.0.1]: https://github.com/thingineeer/claude-session-tools/releases/tag/v1.0.1
[1.0.0]: https://github.com/thingineeer/claude-session-tools/releases/tag/v1.0.0
