# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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

[1.0.0]: https://github.com/thingineeer/claude-config/releases/tag/v1.0.0
