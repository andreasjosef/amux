# Pipeline status reads GitHub, with no per-project config yet

Pipeline status gets CI and deploy states only from GitHub — the check runs and commit
statuses on a commit — rather than from each host's own CLI or API (Vercel, Railway, …).
Hosts with a GitHub integration already post their deploy states there, so one
authenticated `gh` covers every project without per-host tokens, CLIs or config, and a
new host shows up by itself. The cost is that only what a host reports to GitHub is
visible: a deploy that isn't tied to a commit, or a host without the integration, doesn't
appear.

For the same reason there is no `.amux/` config: everything is detected (GitHub remote,
default branch, current branch) and every check and status on the commit is shown, with
names shortened by rule. A per-project file to rename or hide marks waits until a project
actually needs one.
