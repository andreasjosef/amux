# amux

A `tmux` session launcher tailored for an AI-assisted coding workflow. One command per
project directory opens (or re-attaches to) a 4-window session:

| # | Window   | Auto-launches         |
|---|----------|------------------------|
| 0 | `SHAPE`  | `claude --permission-mode auto` — plan/spec/grill |
| 1 | `BUILD`  | `opencode` — implement tickets |
| 2 | `INSPECT` | plain shell — inspect code, nothing auto-run |
| 3 | `SERVER` | `pnpm dev` |

The session is named after the project directory (`.` → `_`). Running `amux` again in
the same project attaches to the existing session instead of rebuilding it.

## Install

Symlinked into `~/.local/bin`:

```sh
ln -sf ~/projects/amux/amux ~/.local/bin/amux
```

Editing and committing the script here takes effect immediately — no reinstall step.

## Usage

```sh
cd ~/projects/some-project
amux
```

Pressing `M-.` (Alt-`.`) inside the session prompts `kill session <name>? (y/n)`.
Confirming with `y` gracefully closes the workspace: it sends Ctrl-C to every
window to let `claude`, `opencode`, and `pnpm dev` exit cleanly (releasing e.g. a
dev-server port), waits ~1 second, then kills the session as a safety net.
Declining with `n` (or Escape) leaves the workspace untouched.

## Known limitations

- `SERVER` hardcodes `pnpm dev`. If a project has no `dev` script, that window's command
  fails on launch — you can still use the window manually. A smarter check (detect
  `package.json`'s `dev` script, fall back to an empty shell) is a possible future
  improvement, not yet implemented.
- No project picker yet (unlike `smux-picker` for `smux`).
