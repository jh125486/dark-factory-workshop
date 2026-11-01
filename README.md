# Building With Guardrails: a Semantic Font Renderer in Go

A hands-on companion workshop to the talk *"Use of AI in Industry: The Modern Dark Factory."* Bring whichever AI coding tool you already have. You give it a goal; it plans, implements, reviews its own work and opens a PR, spawning separate subagents for the roles that need isolation. You never drive it step by step: if you catch yourself typing a second prompt mid-task that isn't an answer to its question, something's wrong.

**This workshop is about managing AI, not writing code.** You won't open an IDE or type a line of Go. You manage a team of AI agents from [herdr](https://herdr.dev): your orchestrator runs each Executor and Auditor in its own herdr pane, so you can watch them work, give goals, and review and approve what the AI produces.

**What gets built:** a semantic font renderer in Go. A scoring library, `lexiscore`, scores each word of a text (valence, salience, surprise, certainty); a renderer turns the text into one self-contained HTML page with embedded CSS where each word's weight, size, colour, slant, spacing and opacity follow its scores. HTML + CSS is the output because it is the easiest and most platform independent: you open the result in a browser.

**Platforms:** macOS · Linux · Windows through WSL2 (use the Linux instructions inside WSL2)

## Run of show

| Step | What | Budget |
|------|------|--------|
| Pre-work | [Before the room](#pre-work) | not timed |
| 1 | [Set up your LLM with rules](#1-set-up-your-llm-with-rules) | ~15 min |
| 2 | [Set up your repo with guardrails](#2-set-up-your-repo-with-guardrails) | ~20 min |
| 3 | [Lock the repo with a ruleset](#3-lock-the-repo-with-a-ruleset) | ~5 min |
| 4 | [Build the semantic font renderer](#4-build-the-semantic-font-renderer) | ~45 min |
| 5 | [Optimize](#5-optimize-optional-stretch-goal) | ~20 min |

Steps 1 to 4 take about 85 minutes; step 5 is an optional stretch goal for anyone who finishes with time left (105 minutes with it), which leaves buffer inside 2 hours. Each step is one copy-pasteable prompt from [`prompts/`](prompts/). You send it, then you wait for a done-state (an open PR, a finished file, or a question), nothing in between. Each budget includes its CI wait (2 to 4 minutes per PR).

## Pre-work

- Clone this workshop repo (`git clone https://github.com/jh125486/dark-factory-workshop`) so you have the prompts.
- Install [herdr](https://herdr.dev), a terminal multiplexer for agents (`brew install herdr` on macOS, `curl -fsSL https://herdr.dev/install.sh | sh` on Linux and WSL2). You will run your AI tool inside it for the whole workshop.
- Install your AI coding tool (any tool works if it can run subagents, use a remote MCP server and run in a terminal) and launch it on its top/frontier model; the orchestrator is the Planner. You need a plan with room for a long agent session; if you only have one model, run everything on it and skip the tier split.
- Install the GitHub MCP for your tool (follow its docs) with the ruleset tools enabled and a classic token with the `repo` and `workflow` scopes and a 1-day expiry. Check it by asking your tool to list your repos and name its ruleset tools; don't let it create anything. Then check the rest: `go version && cc --version && herdr --version && git config user.email`.
- Your terminal can already `git push` feature branches (a token credential needs the `workflow` scope because PR #1 adds `.github/workflows`; or use SSH), and git has `user.name` and `user.email`.
- Install the latest Go 1.27.x, git and a C compiler (`build-essential` on Ubuntu/WSL2, where the race detector needs one; Xcode command line tools on macOS). The AI installs the rest. (The agent can't type a sudo password, so install these yourself.) Make sure `$(go env GOPATH)/bin` and `~/.local/bin` are on your `PATH`.
- Warm the caches so the room doesn't all download at once: run `go run github.com/golangci/golangci-lint/v2/cmd/golangci-lint@v2.15.0 version`, `go run honnef.co/go/tools/cmd/staticcheck@v0.8.2 -version` and `go run golang.org/x/vuln/cmd/govulncheck@v1.9.0 -version`.
- Create an empty working directory next to (not inside) your workshop clone, `mkdir dark-factory-lab && cd dark-factory-lab`, and launch your tool there, inside a herdr pane. Limit it to that directory plus the tool locations the AI writes to (package manager, Go caches, `~/go/bin`, worktrunk config), and turn off per-command approval prompts, per your tool's docs (helpers will walk the room).

> **Why no devcontainer?** A devcontainer is normally the right way to isolate an agent, but it needs a container runtime, which is too much for a 2-hour workshop. Instead this workshop relies on scoped directory access, a revocable token and PR review. No production credentials, ever, on a workshop laptop.

## 1 Set up your LLM with rules

Keep your tool running in its herdr pane for the rest of the workshop; open a second pane when you want to run a command yourself or watch a subagent.

**Give it this one prompt:** [`prompts/01-write-agents-md.md`](prompts/01-write-agents-md.md)

The prompt carries the policy; the AI writes it into `AGENTS.md` (and links the filename your tool reads to it, if that's a different one). It covers the orchestrator/Planner, Executor and Auditor roles (subagents nest at most two levels: orchestrator, subagent, tool caller; never a subagent spawning a subagent), model tiers (frontier Planner and Auditor, mid-tier Executor, cheap model for mechanical setup), worktrees only through [worktrunk](https://worktrunk.dev) (`wt`), Go testing rules, GitHub only through the MCP, protected files, and the retry cap. It also tells the orchestrator to run each Executor and Auditor as its own agent in a herdr pane. The AI authors the rules; you read them and approve.

> ✅ **Checkpoint:** `AGENTS.md` exists, you've read it, and it covers the roles and rules above. (If symlinks fail, the prompt falls back to a copy.)

## 2 Set up your repo with guardrails

**Give it this one prompt:** [`prompts/02-setup-repo.md`](prompts/02-setup-repo.md)

The AI installs [`task`](https://taskfile.dev), `wt` and `benchstat`, creates your own public repo through the MCP, and on a feature branch writes the Go module, a Taskfile with one `gate` task (build, vet, golangci-lint, staticcheck, govulncheck, race tests, a 90% coverage check, each tool at a pinned version), the lint config, a CI workflow that only runs `task gate` in a job named `guardrails`, and a Dependabot config. Then it opens PR #1 and stops.

**You review and merge PR #1 in the browser.** The Taskfile is the honesty control for everything that follows: from now on the AI needs your sign-off to change it. Before you merge, wait for the `guardrails` check to go green; if it's red, tell the agent to fix it on the same PR. Then move on to step 3.

## 3 Lock the repo with a ruleset

**Give it this one prompt (after merging PR #1):** [`prompts/03-ruleset.md`](prompts/03-ruleset.md)

The AI creates the ruleset through the MCP and reads it back to you. It waits for the merge so the ruleset doesn't get in the way of PR #1. Verify in the browser (repo Settings, Rules):

- a pull request is required before merging
- the `guardrails` check is required
- nobody can bypass
- enforcement is Active

If the ruleset is wrong, fix or delete it in the browser. Dependabot PRs will appear later; ignore them.

## 4 Build the semantic font renderer

**Give it this one prompt:** [`prompts/04-build.md`](prompts/04-build.md)

Two parts, one prompt. First the AI proposes `spec/target.html`: one sentence where each word carries inline styles driven by its meaning, with a short comment explaining the mapping. It opens a small PR and gives you the file path. Open it in a browser (on WSL2, run `explorer.exe .` in that folder); if you like it, **merge it (merging is the approval)**, then tell the agent "merged" and it carries on.

Then it builds the renderer itself: a Planner task list, a mid-tier Executor that implements it with its own tests (90%+ coverage), `task gate` run by the orchestrator, an Auditor review, looping until its output for the target sentence correctly matches `spec/target.html` (the same relative styling decisions, not byte equality). That match is the stop condition; the gate must also be green to open the PR. You see a PR when it's done, or a question if it's stuck. If it asks because it hit its retry cap, answer with one of: accept the gaps and open the PR, allow one more round, or loosen the match to the few most emphasized words. If you're past minute 75 of the workshop, tell it to use two axes (valence and salience) and three style properties. Out of time? Review whatever PR or question is open; the PR is the deliverable.

Review the PR like any other: the Taskfile, lint config, CI, Dependabot config, `AGENTS.md` and `spec/target.html` should be untouched, and the tests should assert behavior rather than copy the code's output. Run the build and run commands from the PR description, open `spec/out.html` next to `spec/target.html`, and compare. Merge it yourself; the AI never does.

> **Watch:** the AI writes the tests, so the Auditor and you are the check that they say something. A test that passes without the feature working is the failure to look for.

## 5 Optimize (optional stretch goal)

Skip this if you're behind. **Give it this one prompt:** [`prompts/05-optimize.md`](prompts/05-optimize.md)

"Optimize" means both dimensions `testing.B` reports: time **and** allocations. The prompt asks the AI to write the benchmark first; then the orchestrator captures its own baseline and runs `benchstat` after each round; the Executor makes one change per round, and a round that regresses or breaks a test is reverted. The Executor never certifies its own improvement. One PR at the end.

The numbers below are illustrative (one presenter's run on Apple-silicon hardware, not a spec); expect the same shape, not these values:

| Round | Change | ns/op | B/op | allocs/op |
|---|---|---|---|---|
| 0 | Map lookups per word, regex tokenizer, a new slice per call | ~380000 | ~210000 | ~4100 |
| 1 | Precomputed dictionary as a slice-backed hash, hand-rolled scanner | ~95000 | ~48000 | ~900 |
| 2 | Single preallocated result slice, reused window buffer | ~61000 | ~9600 | ~2 |

## Wrap-up

- Revoke the GitHub token (and any other token you created for this workshop).
- Remove the `worktree-path` line from your worktrunk user config (`wt config show` prints the file).
- The PR is the deliverable, not the transcript.

---

**For helpers:** the GitHub MCP server URL is `https://api.githubcopilot.com/mcp/x/all` (all toolsets); the ruleset tools live in the `governance` toolset, so a default install may lack them. Use a classic token with `repo` and `workflow`: a fine-grained token limited to selected repositories can't cover a repo that doesn't exist yet.

Companion workshop to *"Use of AI in Industry: The Modern Dark Factory."* Bring your own guardrails.
