# amux

A tmux-based project workspace for AI-assisted coding: one workspace per project, made up
of workflow windows you move through plus runners that keep supporting processes running
alongside them.

## Language

### Core concepts

**Workspace**:
The complete per-project environment amux creates and manages. One per project directory;
re-running amux reattaches to the existing Workspace instead of recreating it.
_Avoid_: Session (the tmux mechanism the Workspace is built on, not the domain concept itself)

**Workflow window**:
A window you actively move between while doing the work of coding. Distinct from a Runner,
which isn't a step you move through.
_Avoid_: Stage, Pane

**Runner**:
A window dedicated to a long-running background process that supports the workflow but
isn't itself a step in it.
_Avoid_: Infra window, Utility window

### Workflow windows

**Shape**:
The workflow window for planning and specifying work before it's built.

**Build**:
The workflow window for implementing planned work.

**Inspect**:
The workflow window for manually browsing and reviewing code, with nothing run automatically.
_Avoid_: Review

### Runners

**Server**:
The runner that keeps the project's dev server running.
