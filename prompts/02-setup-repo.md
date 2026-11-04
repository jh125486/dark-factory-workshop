# Step 2 — the one prompt the human gives. Ends with an open PR #1 that you review and merge.

```
Ask me for a repo name, then create it (public, auto-deleting merged
branches), connect this directory to it, and open PR #1 with a Go
module, AGENTS.md, a Taskfile whose one `gate` task runs build, vet,
golangci-lint, staticcheck, govulncheck (all pinned, not installed),
race tests and a 90% coverage check, CI whose one job, `guardrails`,
runs `task gate`, and weekly Dependabot. Worktrees go under
`.worktrees/`. You may edit protected files until PR #1 merges. Wait for
CI, then stop.
```
