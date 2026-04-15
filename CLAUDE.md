# CLAUDE.md

このファイルは Claude Code が omarchy-coding-agent-launcher を触るときに参照する開発コンテキスト。

## プロジェクトの目的

Claude Code / Codex / Gemini CLI / OpenCode などの terminal coding agent を、omarchy / Hyprland 上でプロジェクト単位に素早く切り替えるランチャー。

以前は Claude Code 専用の `claude-launcher` だったが、現在は agent 非依存の `coding-agent-launcher` として設計する。GitHub: https://github.com/komagata/omarchy-coding-agent-launcher

## アーキテクチャ

- **1 キーバインド（Super+I）** → walker popup でプロジェクト選択 → tmux pane で選択中 agent が起動
- **単一 tmux セッション** `coding-agents` を維持し、1 プロジェクト = 1 pane で管理
- 初回: terminal を起動して session attach / 2回目以降: 同じ terminal 内に pane 追加 + `hyprctl` でフォーカス
- 起動 agent は `CODING_AGENT_LAUNCHER_AGENT` で固定選択。未設定時は `claude`
- プロジェクトごとの agent は `<project>/.agents/agent` で上書きできる。worktree は自身の `.agents/agent` がなければ親 project の設定を継承する
- agent 固有の会話履歴・認証・model 設定は各 CLI 側に任せる

## ディレクトリレイアウト前提（固定 depth=2）

```text
$CODING_AGENT_LAUNCHER_WORKS_DIR/<namespace>/<name>/
```

例: `~/Works/komagata/siro-pc/`, `~/Works/fjordllc/bootcamp/`

フラット（depth=1）はサポートしない。

## 環境変数

| 変数 | デフォルト |
|---|---|
| `CODING_AGENT_LAUNCHER_AGENT` | `claude` |
| `CODING_AGENT_LAUNCHER_WORKS_DIR` | `$HOME/Works` |
| `CODING_AGENT_LAUNCHER_DEFAULT_NS` | 未設定（未設定時は `ns/name` 必須）|
| `CODING_AGENT_LAUNCHER_TERMINAL` | `$TERMINAL` → `alacritty` |
| `CODING_AGENT_LAUNCHER_SESSION` | `coding-agents` |
| `CODING_AGENT_LAUNCHER_AGENT_ARGS` | 全 agent 共通の追加引数 |
| `CODING_AGENT_LAUNCHER_CLAUDE_ARGS` | `claude` 専用の追加引数 |
| `CODING_AGENT_LAUNCHER_CODEX_ARGS` | `codex` 専用の追加引数 |
| `CODING_AGENT_LAUNCHER_GEMINI_ARGS` | `gemini` 専用の追加引数 |
| `CODING_AGENT_LAUNCHER_OPENCODE_ARGS` | `opencode` 専用の追加引数 |
| `CODING_AGENT_LAUNCHER_DEBUG` | 未設定（`1` で `/tmp/coding-agent-launcher.log` に出力）|

## 既知のハマりどころ

- **完全リネーム方針**: 旧 `claude-launcher` コマンド、旧 `CLAUDE_LAUNCHER_*` 環境変数の互換 shim は残さない。
- **worktree 保存先**: 新規は `.agents/worktrees/<name>`。既存 `.claude/worktrees` は Claude Code 互換のため移動・削除しない。見つかった場合は `project [name @claude]` として開けるようにする。
- **agent activity 判定**: `pstree` で選択中 agent のプロセス名を探す。子 `bash` があれば `working`、なければ `idle` とみなす。
- **walker の選択結果プレフィックス除去**: `${var#??}` はマルチバイト文字と相性が悪い。`${rel#● }` などで明示的に除去すること。

## ファイル構成

```text
omarchy-coding-agent-launcher/
├── README.md
├── LICENSE
├── CLAUDE.md
├── AGENTS.md
├── bin/
│   └── coding-agent-launcher
├── install.sh
├── examples/
│   └── hypr-keybind.conf
├── docs/
│   ├── screenshot.png
│   └── terminal.png
└── .gitignore
```

## 開発コマンド

```bash
bash -n bin/coding-agent-launcher
bash -n install.sh

CODING_AGENT_LAUNCHER_DEBUG=1 ./bin/coding-agent-launcher
tail -f /tmp/coding-agent-launcher.log

./install.sh
```

## 公開スコープの方針

ランチャーは「プロジェクト切替」と「coding agent 起動」に徹する。agent 固有の memory / profile / MCP / model 設定は各 CLI に任せる。

## 残タスク・アイデア

- デモ GIF（asciinema or peek）を docs/ に追加
- install.sh に uninstall オプション
- flat layout（depth=1）のサポート要望が来たら検討
- `CODING_AGENT_LAUNCHER_NEW_PROJECT_HOOK` のようなフック
- walker 以外のランチャー（rofi, fuzzel）対応

## 個人環境との関係

komagata の `~/.local/bin/coding-agent-launcher` はこのリポジトリの `bin/coding-agent-launcher` へのシンボリックリンクにすると、リポジトリ更新が本人環境に即反映される。個人環境固有の設定は `~/.config/hypr/envs.conf` と `~/.bashrc` に環境変数として分離する。
