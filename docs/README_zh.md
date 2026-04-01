🌐 [English](../README.md) | [한국어](README_ko.md) | [日本語](README_ja.md) | [中文](README_zh.md)

# claude-session-tools

> 跨设备保存和恢复 [Claude Code](https://claude.com/claude-code) 会话。再也不会丢失工作上下文。

如果这个插件帮到了你，请给一个 [star](https://github.com/thingineeer/claude-session-tools)。

## 问题

Claude Code 将会话历史存储在本地 `~/.claude/sessions/`。这带来了两个问题：

1. **跨设备**：切换电脑后，上下文就消失了
2. **可靠性**：即使在同一台设备上，`claude --continue` 和 `--resume` 依赖本地会话文件，更新后这些文件可能过期、损坏或丢失 — 最终不得不从头开始重新解释

## 解决方案

**session-saver** 不依赖本地会话历史。它将完整的工作上下文保存为 **git 提交的文件**，并生成一个项目本地的 `/resume-{folder}` 命令，自动拉取并恢复所有内容。

| 命令 | 范围 | 功能 |
|------|------|------|
| `/session-saver:save-session` | 全局（插件） | 保存上下文 + 生成 `/resume-{folder}` + push |
| `/resume-{folder}` | 项目本地（自动生成） | Auto-pull + 恢复完整上下文 |

> **注意**：插件技能按照 Claude Code 规范使用 `/plugin-name:skill-name` 命名空间格式。resume 命令作为项目本地技能生成，因此使用简短的 `/resume-{folder}` 格式。

## 安装

需要 [Claude Code](https://claude.com/claude-code) v1.0.33 或更高版本。

### 方式1：提示词

将以下提示词粘贴到 Claude Code 中 — 它会在保留现有设置的同时自动安装插件：

```
Add the following to ~/.claude/settings.json without removing any existing settings:

In enabledPlugins:
  "session-saver@claude-session-tools": true

In extraKnownMarketplaces:
  "claude-session-tools": { "source": { "source": "github", "repo": "thingineeer/claude-session-tools" } }
```

### 方式2：插件菜单

```
/plugins → Add marketplace → thingineeer/claude-session-tools → Install session-saver
```

安装后重启 Claude Code 即可。

## 使用方法

### 保存（离开前）

```
/session-saver:save-session
```

执行的操作：
1. 将所有未提交的更改按逻辑单元分割提交
2. 清理 worktree 和 auto memory
3. 在 `docs/checkpoints/SESSION-STATE.md` 中保存完整上下文
4. 为项目生成 `.claude/skills/resume-{folder}/SKILL.md`
5. Push 到远程仓库

### 恢复（任何设备，任何时间）

```
/resume-{folder-name}
```

执行的操作：
1. `git fetch` — 如果落后于远程，自动 `git pull`
2. 读取 CLAUDE.md 和 SESSION-STATE.md
3. 读取存档点中记录的 key files
4. 输出包含分支、进度和下一步任务的简报

无需手动 `git pull`。resume 命令会自动处理。

## 支持的场景

| 场景 | 命令 |
|------|------|
| 设备A → 设备B（立即） | 在B上 `/resume-{folder}` — 自动pull |
| 设备A → 几天后 → 设备B | 相同 — 存档点不会过期 |
| 设备A → 几天后 → 设备A | 相同 — 获取最新状态，从git恢复 |
| 设备A → 重启应用 → 设备A | 相同命令正常工作 |

## 工作原理

```
设备A                                       设备B
  |                                           |
  |-- /session-saver:save-session             |
  |     |-- 提交更改                            |
  |     |-- 写入 SESSION-STATE.md              |
  |     |-- 生成 /resume-{folder} 技能          |
  |     |-- push                              |
  |                                           |
  |              git push ────────>           |
  |                                           |
  |                                           |-- /resume-{folder}
  |                                           |     |-- git fetch + auto pull
  |                                           |     |-- 读取 CLAUDE.md
  |                                           |     |-- 读取 SESSION-STATE.md
  |                                           |     |-- 读取 key files
  |                                           |     |-- 输出简报
```

### 保存的内容

| 文件 | 用途 |
|------|------|
| `docs/checkpoints/SESSION-STATE.md` | 存档点 — 分支、进度、key files、下一步任务 |
| `.claude/skills/resume-{folder}/SKILL.md` | 恢复技能 — auto-pull + 上下文重建 |

### 不保存的内容

- 会话对话历史（保留在本地）
- 密钥、凭证、`.env` 文件（设计上排除）
- 自动生成的文件（`node_modules/`、`Derived/`、`build/`）

## 从 v1.0.x 迁移

如果你使用过 v1.0.x，resume 命令生成在 `.claude/commands/resume-{folder}.md`。从 v1.1.0 开始，改为生成在 `.claude/skills/resume-{folder}/SKILL.md`。

在有旧格式命令的项目中运行 `/session-saver:save-session` 会自动迁移到新的技能格式并删除旧命令文件。

## 自定义

Fork 此仓库并修改 `plugins/session-saver/skills/save-session/SKILL.md` 以适配你的工作流：

- 修改 SESSION-STATE.md 模板
- 添加项目特定的构建验证步骤
- 调整提交消息规范

## 贡献

欢迎贡献！请提交 issue 或发起 Pull Request。

1. Fork 此仓库
2. 创建 feature 分支（`git checkout -b feat/my-feature`）
3. 按照 [Conventional Commits](https://www.conventionalcommits.org/) 规范提交
4. Push 到分支并创建 Pull Request

## 许可证

[MIT](../LICENSE)
