# Step 2 — the one prompt the human gives. Ends with an open PR #1 that you review and merge.

```
Per AGENTS.md, you are the orchestrator. This is mechanical setup: no
Executor or Auditor needed: do it yourself, or hand it to a cheap-tier
subagent if your tool can; don't ask me to run commands. Ask me for my GitHub username and a repo name (suggest
`semantic-font`) if you don't know them.

1. Install what's missing: go-task (`task`, not Taskwarrior),
   benchstat and worktrunk
   (`wt`, https://worktrunk.dev). Set worktrunk's `worktree-path` in its
   user config to `{{ repo_path }}/.worktrees/{{ branch | sanitize }}`.
2. With the GitHub MCP, create my repo: public, with an initial README
   commit so `main` exists. Connect this directory to it (`git init`,
   remote, fetch, local `main` tracking origin/main).
3. On a feature worktree made with `wt` (never on main), MOVE AGENTS.md
   and its instruction-file link (if any) into the worktree, and commit them in this PR (move, don't copy,
   so the primary checkout stays clean and can pull main later), and
   write:
   - go.mod (Go 1.27) and a minimal placeholder package with one
     trivial test, so the gate has something real to run (the build
     step replaces it).
   - Taskfile.yml with ONE task, `gate`, running in order: build, vet,
     golangci-lint, staticcheck, govulncheck, tests with `-race`, and
     a coverage check that fails under 90% total. Run every tool on `./...`
     with `go run <package>@<version>` at exactly these versions:
     golangci-lint v2.15.0 (github.com/golangci/golangci-lint/v2/cmd/golangci-lint),
     staticcheck v0.8.2 (honnef.co/go/tools/cmd/staticcheck),
     govulncheck v1.9.0 (golang.org/x/vuln/cmd/govulncheck), so CI and your
     machine use the same versions and go.mod stays clean. No
     benchmarks in the gate; a later step runs those. It must pass on
     what is in this PR.
   - .golangci.yml: a sensible baseline, not everything turned on.
   - .github/workflows/ci.yml: on pull_request, one job named
     `guardrails` that sets up Go 1.27 and task and runs only
     `task gate` (actions/checkout@v7, actions/setup-go@v7,
     go-task/setup-task@v2).
   - .github/dependabot.yml: Go modules and GitHub Actions, weekly.
   - .gitignore: gate.log, bench*.txt, coverage output, .worktrees/,
     spec/out.html.
4. Protected-file rules apply from PR #1's merge onward; until then you
   may edit these files on this PR. Run `task gate`, push the branch, open PR #1 through the MCP with
   the gate output in the description. Wait for CI, confirm the check
   is named exactly `guardrails`, report its result, and stop. Once CI
   is green I review and merge it.
```
