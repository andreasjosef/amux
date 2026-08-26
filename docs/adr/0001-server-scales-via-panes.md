# Server scales via panes, not windows

Today `Server` runs a single dev-server process. When it needs to run more than one
process at once (e.g. separate local/remote processes, or per-environment processes
like dev/prod — that split isn't decided yet), those will be added as additional panes
within the `Server` window rather than as separate top-level Runner windows. This keeps
every other Runner following the same one-window-per-concern pattern the Workflow windows
use, instead of the workspace's window count growing per environment.
