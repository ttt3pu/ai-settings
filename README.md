# ai-settings

複数リポジトリで共有する AI エージェント用 Agent Skills の配信元。Skill 本体は `skills/<name>/SKILL.md` の 1 か所だけに置き、ベンダーごとのマニフェストはそこを指すだけの薄いものにしている。

## 収録 Skill

| Skill | 内容 |
| --- | --- |
| [`enable-library-automerge`](skills/enable-library-automerge/SKILL.md) | ライブラリの Renovate minor/patch 自動マージを、プロダクト経路の回帰テストと CI 通過を条件に有効化する手順 |
| [`shared-testing-conventions`](skills/shared-testing-conventions/SKILL.md) | テスト名を日本語の「◯◯こと」形にする命名規則、テスト対象の選び方、配置とスナップショットの扱い |

## インストール

### Cursor

ルートの `plugin.json` が [Agent Plugins](https://agent-plugins.org) 標準のマニフェストなので、Cursor はこのリポジトリをそのまま Agent Plugin として読む。

- **Team marketplace**: Dashboard → Plugins → Team Marketplaces → Add Marketplace →「Import from Repo」でこのリポジトリの URL を指定する。Marketplace Settings で Auto Refresh を有効にすると push のたびに再インデックスされる（Cursor GitHub App が必要）
- **ローカルで試す**: `~/.cursor/plugins/local` にリポジトリを置くか symlink する

```sh
ln -s /path/to/ai-settings ~/.cursor/plugins/local/ai-settings
```

置いたあと Cursor を再起動するか `Developer: Reload Window` を実行し、Customize → Skills に出ているか確認する。

参考: [Plugins](https://cursor.com/docs/plugins) / [Plugins reference](https://cursor.com/docs/reference/plugins)

### Claude Code

```sh
claude plugin marketplace add ttt3pu/ai-settings
claude plugin install ai-settings@ttt3pu-ai-settings
```

セッション内から実行する場合は `/plugin marketplace add ttt3pu/ai-settings` → `/plugin install ai-settings@ttt3pu-ai-settings`。インストール結果に `Run /reload-plugins to activate.` と出たら `/reload-plugins` を実行する。

チーム全員に配りたい場合は、利用側リポジトリの `.claude/settings.json` に書く。

```json
{
  "extraKnownMarketplaces": {
    "ttt3pu-ai-settings": {
      "source": { "source": "github", "repo": "ttt3pu/ai-settings" }
    }
  },
  "enabledPlugins": {
    "ai-settings@ttt3pu-ai-settings": true
  }
}
```

参考: [Create and distribute a plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces)

### Codex

```sh
codex plugin marketplace add ttt3pu/ai-settings
```

マーケットプレイスを追加したあと、ChatGPT デスクトップアプリの Plugins Directory でこのマーケットプレイスを選んでインストールする（`codex plugin marketplace` サブコマンドはカタログの登録・更新用で、インストールはデスクトップアプリ側で行う）。

参考: [Package your plugin](https://developers.openai.com/plugins/build/plugins) / [Build skills](https://developers.openai.com/codex/skills)

### GitHub Copilot

Copilot はプラグインマニフェストを `.plugin/plugin.json` → `plugin.json` → `.github/plugin/plugin.json` → `.claude-plugin/plugin.json` の順、マーケットプレイスマニフェストを `marketplace.json` → `.plugin/marketplace.json` → `.github/plugin/marketplace.json` → `.claude-plugin/marketplace.json` の順で探す。このリポジトリはルート `plugin.json` と `.claude-plugin/marketplace.json` の両方を持っているので、追加ファイルなしでそのまま動く。

```sh
copilot plugin marketplace add ttt3pu/ai-settings
copilot plugin install ai-settings@ttt3pu-ai-settings
```

参考: [Copilot CLI plugin reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-plugin-reference)

### Antigravity (Gemini)

Antigravity のプラグインもルートの `plugin.json` を marker file とし、`skills/<name>/SKILL.md` を自動探索するので、追加ファイルなしで動く。ただし Antigravity には他ツールのようなマーケットプレイスカタログのファイル形式がないため、リポジトリを clone して所定の場所に置く形になる。

全ワークスペースで有効にする場合:

```sh
git clone https://github.com/ttt3pu/ai-settings ~/.gemini/config/plugins/ai-settings
```

特定のワークスペースだけで有効にする場合は、そのワークスペース直下の `.agents/plugins/` か `_agents/plugins/` にプラグインフォルダを置く。

`agy` CLI があれば、clone したディレクトリを指定してインストールと検証ができる。

```sh
agy plugin validate /path/to/ai-settings
agy plugin install /path/to/ai-settings
```

参考: [Plugins | Google Antigravity Docs](https://antigravity.google/docs/ide/plugins/)

## Skill を追加する

1. `skills/<name>/SKILL.md` を作る。`<name>` は小文字・数字・ハイフンのみ、64 文字以内
2. frontmatter の `name` をディレクトリ名と完全に一致させる。ファイル名も `SKILL.md` の完全一致でなければ読み込まれない
3. `description` に「何をするか」と「いつ使うか」の両方を書く。改行を含めず簡潔にし、トリガーになる語を前に置く（Codex は初期スキル一覧の予算を超えると description を短縮する）
4. マニフェストへの追記は不要。5 つのマニフェストはすべて `skills/` ディレクトリ全体を指しているか、フォルダ探索に任せている

追加ファイル（`examples.md`、`scripts/`、`references/`、`assets/`）は Skill ディレクトリ内に置き、`SKILL.md` から相対パスで参照する。

### 注意点

- 共有 Skill の名前は、利用側リポジトリのプロジェクト固有 Skill と衝突しないものにする。Codex は同名 Skill をマージせず両方をセレクタに出す（`testing` を `shared-testing-conventions` に改名しているのはこの理由）
- `paths` は Cursor 専用の frontmatter フィールド。他ツールでは無視されるので、必須の挙動をこれに依存させない
- `.agents/plugins/` は Codex のマーケットプレイスカタログの置き場所と、Antigravity がワークスペースのプラグインフォルダを探す場所が重なっている。ここに置いてよいのは `marketplace.json` だけで、プラグインフォルダを増やさない
- リリース時は `plugin.json`、`.claude-plugin/plugin.json`、`.codex-plugin/plugin.json` の `version` を揃えて上げる。Claude Code と Codex は `version` が変わらないとキャッシュを更新しない

## ファイル構成

```
plugin.json                        # Agent Plugins 標準。Cursor / Copilot / Antigravity がこれを読む
.claude-plugin/plugin.json         # Claude Code 用プラグインマニフェスト
.claude-plugin/marketplace.json    # Claude Code 用マーケットプレイスカタログ（Copilot の fallback 探索先でもある）
.codex-plugin/plugin.json          # Codex 用プラグインマニフェスト
.agents/plugins/marketplace.json   # Codex 用マーケットプレイスカタログ
skills/<name>/SKILL.md             # Skill 本体。内容はここにしか書かない
```
