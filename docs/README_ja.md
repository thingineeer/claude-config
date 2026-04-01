🌐 [English](../README.md) | [한국어](README_ko.md) | [日本語](README_ja.md) | [中文](README_zh.md)

# claude-session-tools

> [Claude Code](https://claude.com/claude-code)のセッションをデバイス間で保存・復元します。作業コンテキストをもう二度と失いません。

このプラグインが役に立ったら、[star](https://github.com/thingineeer/claude-session-tools)をお願いします。

## 課題

Claude Codeはセッション履歴を`~/.claude/sessions/`にローカル保存します。これにより2つの問題が発生します：

1. **クロスデバイス**: コンピュータを切り替えるとコンテキストが消えます
2. **信頼性**: 同じデバイスでも`claude --continue`や`--resume`はローカルセッションファイルに依存しているため、アップデート後にファイルが古くなったり、破損したり、失われたりする可能性があります — 結局すべてを最初から説明し直すことになります

## 解決策

**session-saver**はローカルセッション履歴に依存しません。作業コンテキスト全体を**gitコミットされたファイル**として保存し、自動でpullして復元するプロジェクトローカルの`/resume-{folder}`コマンドを生成します。

| コマンド | スコープ | 機能 |
|----------|----------|------|
| `/session-saver:save-session` | グローバル（プラグイン） | コンテキスト保存 + `/resume-{folder}`生成 + push |
| `/resume-{folder}` | プロジェクトローカル（自動生成） | Auto-pull + コンテキスト全体の復元 |

> **注意**: プラグインスキルはClaude Codeの規約に従い`/plugin-name:skill-name`形式でネームスペースされます。resumeコマンドはプロジェクトローカルスキルとして生成されるため、短い`/resume-{folder}`形式を使用します。

## インストール

[Claude Code](https://claude.com/claude-code) v1.0.33以降が必要です。

### 方法1: プロンプト

以下のプロンプトをClaude Codeに貼り付けてください — 既存の設定を保持しながら自動的にプラグインをインストールします：

```
Add the following to ~/.claude/settings.json without removing any existing settings:

In enabledPlugins:
  "session-saver@claude-session-tools": true

In extraKnownMarketplaces:
  "claude-session-tools": { "source": { "source": "github", "repo": "thingineeer/claude-session-tools" } }
```

### 方法2: プラグインメニュー

```
/plugins → Add marketplace → thingineeer/claude-session-tools → Install session-saver
```

インストール後、Claude Codeを再起動すれば完了です。

## 使い方

### 保存（離席前）

```
/session-saver:save-session
```

実行される処理：
1. 未コミットの変更をすべて論理単位で分割してコミット
2. worktreeとauto memoryのクリーンアップ
3. `docs/checkpoints/SESSION-STATE.md`にコンテキスト全体を保存
4. プロジェクトに`.claude/skills/resume-{folder}/SKILL.md`を生成
5. リモートにpush

### 復元（どのデバイスでも、いつでも）

```
/resume-{folder-name}
```

実行される処理：
1. `git fetch` — リモートより遅れている場合は自動`git pull`
2. CLAUDE.mdとSESSION-STATE.mdを読み込み
3. セーブポイントに記録されたkey filesを読み込み
4. ブランチ、進捗状況、次のタスクをブリーフィングとして出力

手動で`git pull`する必要はありません。resumeコマンドが自動で処理します。

## 対応シナリオ

| シナリオ | コマンド |
|----------|----------|
| デバイスA → デバイスB（即座に） | Bで`/resume-{folder}` — 自動pull |
| デバイスA → 数日後 → デバイスB | 同様 — セーブポイントに有効期限はありません |
| デバイスA → 数日後 → デバイスA | 同様 — 最新を取得してgitから復元 |
| デバイスA → アプリ再起動 → デバイスA | 同じコマンドで動作 |

## 仕組み

```
デバイスA                                    デバイスB
  |                                           |
  |-- /session-saver:save-session             |
  |     |-- 変更をコミット                      |
  |     |-- SESSION-STATE.md作成               |
  |     |-- /resume-{folder}スキル生成          |
  |     |-- push                              |
  |                                           |
  |              git push ────────>           |
  |                                           |
  |                                           |-- /resume-{folder}
  |                                           |     |-- git fetch + auto pull
  |                                           |     |-- CLAUDE.md読み込み
  |                                           |     |-- SESSION-STATE.md読み込み
  |                                           |     |-- key files読み込み
  |                                           |     |-- ブリーフィング出力
```

### 保存されるもの

| ファイル | 用途 |
|----------|------|
| `docs/checkpoints/SESSION-STATE.md` | セーブポイント — ブランチ、進捗、key files、次のタスク |
| `.claude/skills/resume-{folder}/SKILL.md` | 復元スキル — auto-pull + コンテキスト再構築 |

### 保存されないもの

- セッション会話履歴（ローカルに保持）
- シークレット、認証情報、`.env`ファイル（意図的に除外）
- 自動生成ファイル（`node_modules/`、`Derived/`、`build/`）

## v1.0.xからのマイグレーション

v1.0.xを使用していた場合、resumeコマンドは`.claude/commands/resume-{folder}.md`として生成されていました。v1.1.0からは`.claude/skills/resume-{folder}/SKILL.md`として生成されます。

旧形式のコマンドがあるプロジェクトで`/session-saver:save-session`を実行すると、自動的に新しいスキル形式にマイグレーションし、レガシーコマンドファイルを削除します。

## カスタマイズ

このリポジトリをforkして`plugins/session-saver/skills/save-session/SKILL.md`をワークフローに合わせて修正してください：

- SESSION-STATE.mdテンプレートの変更
- プロジェクト固有のビルド検証ステップの追加
- コミットメッセージ規約の調整

## コントリビューション

コントリビューションを歓迎します！issueを開くか、プルリクエストを送ってください。

1. リポジトリをfork
2. featureブランチを作成（`git checkout -b feat/my-feature`）
3. [Conventional Commits](https://www.conventionalcommits.org/)に従ってコミット
4. ブランチにpushしてPull Requestを作成

## ライセンス

[MIT](../LICENSE)
