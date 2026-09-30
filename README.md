# amux

A `tmux` session launcher tailored for an AI-assisted coding workflow. One command per
project directory opens (or re-attaches to) a 4-window session:

| # | Window   | Auto-launches         |
|---|----------|------------------------|
| 0 | `CLAUDE` | `claude --permission-mode auto` |
| 1 | `NVIM`   | `nvim` |
| 2 | `TERM`   | plain shell |
| 3 | `SERVER` | `pnpm dev` |

`amux old` opens the original layout instead:

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

Pressing `M-o` (Alt-`o`) toggles lazygit in an on-demand `LGIT` window. From any window
it jumps to `LGIT`, opening it (in the current pane's directory) if it isn't there yet;
from `LGIT` it jumps back to the window you came from. With the matching Neovim setup,
lazygit's `e` opens the file in `INSPECT`'s Neovim and switches there, so reviewing is
`M-o` → `e` → `M-o`. Quitting lazygit (`q`) closes the window.

`M-p` does the same for [gh-dash](https://github.com/dlvhdr/gh-dash) in a `PRS` window:
the repo's pull requests and issues, to view, check out, comment, approve, merge or close.
Install it once with `gh extension install dlvhdr/gh-dash`.

### Pipeline status

For a project on GitHub, the right of the status line shows the **pipeline status** of the
default branch: its CI checks and deploys, as GitHub records them for the branch's latest
pushed commit. `M-c` (Alt-`c`) toggles the session to the current branch's pushed commit
and back.

```
main  ✓ CI  ✓ api  ✓ landing  ✓ app     # default branch (production deploys)
wkd-31  ⟳ CI  ✓ app  ✓ landing          # after M-c: current branch (previews)
```

- `CI` is every GitHub check run on the commit (the latest attempt of each; skipped ones
  ignored), folded into one mark: ✗ if any failed, else ⟳ if any is still running, else ✓.
- Every other mark is a commit status a service posted back to GitHub — e.g. Vercel's and
  Railway's deploys — named without the service and repo prefix (`Vercel – weeklydrip-app`
  → `app`). ✓ means that commit deployed, not that the site is up right now.
- A new push starts over at ⟳ for the new commit; a service that didn't build a commit has
  no mark for it. A branch not on GitHub yet shows `unpushed`.
- Ticket-style branch names shorten to the ticket id (`wkd-31-the-budget-…` → `wkd-31`).

It's read with `gh` (so it uses your `gh auth`), refreshed at most every 30 seconds, with
both views fetched together so `M-c` switches instantly — about 4 API calls per 30s per
workspace. Projects without a GitHub remote show nothing. The segment is put in front of
your global `status-right`, which stays as it is. The work is done by
`amux-pipeline-status`, which `amux` finds next to its real (symlinked) file, so the
install step above is unchanged.

## Known limitations

- `SERVER` hardcodes `pnpm dev`. If a project has no `dev` script, that window's command
  fails on launch — you can still use the window manually. A smarter check (detect
  `package.json`'s `dev` script, fall back to an empty shell) is a possible future
  improvement, not yet implemented.
- Pipeline status has no per-project config: every check and commit status on the commit
  is shown. A `.amux/` file to rename or hide marks is the obvious extension once a project
  needs one (see `docs/adr/0002-pipeline-status-reads-github.md`).
- Pipeline status and the `M-c` binding are set when the workspace is created; a workspace
  created before this feature needs recreating (or the two `tmux set-option` calls from
  `amux` run by hand) to get them.
- No project picker yet (unlike `smux-picker` for `smux`).
- tmux only refreshes a *new* session's environment from whatever process first started
  its server — a session created later on an already-running server (e.g. left over from
  an earlier sandboxed tool invocation) silently inherits that frozen environment instead
  of the current shell's. `amux` pins `HOME` explicitly to guard against this (it's what
  breaks `claude`'s config lookup when stale), but doesn't pin the rest of the
  environment, so a stale server could still leak other vars (e.g. `PATH`) into a new
  workspace. If a launched tool behaves as if it's on a different machine, check for a
  leftover `tmux` server with `tmux ls` and `tmux kill-server` it.
