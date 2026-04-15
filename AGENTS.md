# Repository Guidelines

## Project Structure & Module Organization

This repository is a small Bash utility for omarchy/Hyprland users.

- `bin/coding-agent-launcher` is the main executable. Keep launcher behavior, tmux logic, walker integration, agent selection, and project/worktree handling here unless a split becomes clearly useful.
- `install.sh` is the interactive installer that copies the launcher and optionally appends the Hyprland keybind.
- `README.md` is the user-facing documentation.
- `CLAUDE.md` contains maintainer context.
- `examples/hypr-keybind.conf` provides a sample Hyprland binding.
- `docs/` contains screenshots used by the README.

The launcher assumes projects live at `$CODING_AGENT_LAUNCHER_WORKS_DIR/<namespace>/<name>/`. Worktrees live under `.agents/worktrees/`.

## Build, Test, and Development Commands

There is no build step; scripts run directly with Bash.

```bash
bash -n bin/coding-agent-launcher
bash -n install.sh
```

Run syntax checks before committing.

```bash
CODING_AGENT_LAUNCHER_DEBUG=1 ./bin/coding-agent-launcher
tail -f /tmp/coding-agent-launcher.log
```

Use these for local manual testing on a system with `walker`, `hyprctl`, `tmux`, a supported terminal, and one coding agent CLI.

```bash
./install.sh
```

Runs the installer against the local user environment. It can write to `~/.local/bin` and `~/.config/hypr/bindings.conf`.

## Coding Style & Naming Conventions

Use Bash with `set -euo pipefail`, two-space indentation, and small functions named in `snake_case`. Prefer `local` variables inside functions. Quote variable expansions unless word splitting is intended. Keep user-facing strings concise and route optional diagnostics through `log()`.

Environment variables use the `CODING_AGENT_LAUNCHER_*` prefix. Supported agent identifiers are `claude`, `codex`, `gemini`, and `opencode`. Project identifiers are relative paths such as `komagata/app` or worktree labels like `komagata/app [feature]`.

## Testing Guidelines

No automated test framework is currently configured. At minimum, run `bash -n` on changed scripts and manually exercise affected paths through `./bin/coding-agent-launcher` with debug logging enabled. For installer changes, test on a disposable Hyprland config or inspect prompts carefully before accepting writes.

## Commit & Pull Request Guidelines

Recent commits use short imperative subjects, for example `Add worktree support, ghostty support, and various UX improvements`. Keep subjects direct and under about 72 characters when practical.

Pull requests should describe the user-visible behavior change, list manual validation performed, and include screenshots or terminal output when UI, walker, tmux layout, or installer prompts change. Discuss large scope changes, especially flat directory layout support or alternate launcher backends, before implementation.

## Agent-Specific Instructions

- このリポジトリでは、ユーザーとのやり取りは日本語で行う。
- コード、コマンド、エラーメッセージ、ファイル名は原文のまま扱う。
- 公開ドキュメント本文は既存ファイルの言語に合わせる。`README.md` は英語を維持する。
