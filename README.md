# Building With Guardrails: a Semantic Font Renderer in Go

A hands-on companion workshop to the talk *"Use of AI in Industry: The Modern Dark Factory."* Bring whichever AI coding tool you already have. You give it a goal; it plans, implements, reviews its own work and opens a PR, spawning separate subagents for the roles that need isolation. You never drive it step by step: if you catch yourself typing a second prompt mid-task that isn't an answer to its question, something's wrong.

**This workshop is about managing AI, not writing code.** You won't open an IDE or type a line of Go. You manage a team of AI agents from [herdr](https://herdr.dev): your orchestrator runs each Executor and Auditor in its own herdr pane, so you can watch them work, give goals, and review and approve what the AI produces.

**What gets built:** a semantic font renderer in Go. A scoring library, `lexiscore`, scores each word of a text (valence, salience, surprise, certainty), with valence from the VADER lexicon; a renderer turns the text into one self-contained HTML page with embedded CSS where each word's weight, size, color, slant, spacing and opacity follow its scores. HTML + CSS is the output because it is the easiest and most platform independent: you open the result in a browser.

**The four scores.** Only valence is sentiment in the strict sense; the other three are about emphasis and tone. The AI decides how each one maps to style when it writes `spec/target.html`.

| Score | Question it answers | Low → high example | Where it comes from |
|---|---|---|---|
| **Valence** | How negative or positive is the word? | "grief" → "joy" | The VADER lexicon: human ratings from −4 to +4 for about 7,200 words and 300 emoticons ("grief" −2.2, "joy" +2.8) |
| **Salience** | How much does the word matter in the sentence? | "the" → "danger" | Derived by the AI, e.g. function words low, emotionally loaded words high |
| **Surprise** | How unexpected is it? | "usually" → "suddenly" | Derived by the AI, e.g. words that signal the unexpected, or a word right after "but" |
| **Certainty** | How hedged or confident is it? | "perhaps" → "definitely" | Derived by the AI, e.g. hedges like "maybe" vs boosters like "absolutely" |

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

### Setup, then the factory

Only step 4 is the dark factory. The steps before it are one-time setup, and step 5 is the same factory run again.

| Step | Kind | How often |
|------|------|-----------|
| 1 | **LLM setup**: the AI writes its own rules (`AGENTS.md`) | once per project, reused by every later run |
| 2 | **Repo setup**: the gate (Taskfile, lint, CI) and its protected files | once per repo |
| 3 | **Lock**: a ruleset requires the gate on every merge | once per repo, kept separate on purpose (see below) |
| 4 | **The dark factory**: goal in, reviewed PR out | every feature; this is what you would keep doing |
| 5 | **Stretch goal**: another factory run (optimize), time permitting | a normal factory task; only optional here because of the clock |

Step 3 is separate for two reasons. It is the safety step, since the ruleset is what stops the agent from pushing to `main` or merging past a red check. It isn't a hard boundary: the workshop token can still edit rulesets, so `AGENTS.md` protects repo settings and rulesets and the Auditor rejects any change to them. And it has to wait: the `guardrails` check it requires doesn't exist until PR #1's CI has run, and a ruleset in place earlier would block PR #1 itself.

## Pre-work

- Clone this workshop repo (`git clone https://github.com/jh125486/dark-factory-workshop`) so you have the prompts.
- Install [herdr](https://herdr.dev), a terminal multiplexer for agents (`brew install herdr` on macOS, `curl -fsSL https://herdr.dev/install.sh | sh` on Linux and WSL2). You will run your AI tool inside it for the whole workshop.
- Install [worktrunk](https://worktrunk.dev) (`wt`), which the AI uses for every worktree: `brew install worktrunk` (macOS and Linux) or `cargo install worktrunk`. The AI reads `wt --help` itself, so you don't need to learn it.
- Install your AI coding tool (any tool works if it can run subagents and run in a terminal) and launch it on one of its strongest models (not necessarily the single most expensive one); the orchestrator is the Planner and runs the whole loop. The AI picks the models for the other roles from your tool's model list and records them in `AGENTS.md`. You need a plan with room for a long agent session (one measured run of steps 1 to 5 under Codex on a Plus plan used roughly half of a 5-hour usage window). If your tool asks whether to trust hooks, decline: the workshop doesn't need any. If you only have one model, run everything on it and skip the tier split.
- Install the [GitHub CLI](https://cli.github.com) (`gh`, version 2.99.0 or newer; `gh --version`; older versions lack `--attach`, which step 4 uses to attach screenshots if you install Playwright). Create a classic token with the `repo` and `workflow` scopes and a 1-day expiry, and export it as `GH_TOKEN` in the shell you launch your tool from (skip `gh auth login`, so the short-lived token is the only credential). Run `gh auth setup-git` so `git push` uses that token too (this changes your global git credential helper for GitHub; the wrap-up says how to undo it). Check it with `gh api user --jq .login`, then check the rest: `go version && cc --version && herdr --version && wt --version && git config user.email`.
- Your terminal can already `git push` feature branches (a token credential needs the `workflow` scope because PR #1 adds `.github/workflows`; or use SSH), and git has `user.name` and `user.email`.
- Install the latest Go 1.27.x, git and a C compiler (`build-essential` on Ubuntu/WSL2, where the race detector needs one; Xcode command line tools on macOS). The AI installs the rest. (The agent can't type a sudo password, so install these yourself.) Make sure `$(go env GOPATH)/bin` and `~/.local/bin` are on your `PATH`.
- Warm the caches so the room doesn't all download at once: run `go run github.com/golangci/golangci-lint/v2/cmd/golangci-lint@latest version`, `go run honnef.co/go/tools/cmd/staticcheck@latest -version` and `go run golang.org/x/vuln/cmd/govulncheck@latest -version`.
- Optional: install [Playwright](https://playwright.dev) and its browser (`npx playwright install --with-deps chromium`; on Linux and WSL2 this runs `sudo`) if you want screenshots in your step 4 PR. Without it the agent skips them.
- Create an empty working directory next to (not inside) your workshop clone, `mkdir dark-factory-lab && cd dark-factory-lab`, and launch your tool there, inside a herdr pane. Limit it to that directory plus the tool locations the AI writes to (package manager, Go caches, `~/go/bin`, worktrunk config), and turn off per-command approval prompts, per your tool's docs (helpers will walk the room).

> **Why no devcontainer?** A devcontainer is normally the right way to isolate an agent, but it needs a container runtime, which is too much for a 2-hour workshop. Instead this workshop relies on scoped directory access, a revocable token and PR review. No production credentials, ever, on a workshop laptop.

## 1 Set up your LLM with rules

### Pick your orchestrator

The model you launch is the orchestrator/Planner. It turns your goal into tasks, delegates the coding and tracks the results. Look for strong planning and instruction following, reliable use of tools and subagents, and enough context to keep the rules, plan and task results in view through a long session.

For this workshop, start with one of your tool's newest strong reasoning models below its priciest tier, with high reasoning effort if available. Balance quality against cost, usage limits and waiting time, since this model runs the whole loop. The orchestrator plans and delegates; Executors write code. Step 1 assigns the best coding model below the priciest tier to Executor, the most capable model at high effort in fresh context to Auditor, and the cheapest to Tooler.

Use Step 1's output as your first check: does it preserve the rules and delegate a fresh-context audit without repeated reminders? If you only have one model, use it for all roles and skip the tier split; keep each Executor and Auditor in its own context.

Keep your tool running in its herdr pane for the rest of the workshop; open a second pane when you want to run a command yourself or watch a subagent.

**Give it this one prompt:** [`prompts/01-write-agents-md.md`](prompts/01-write-agents-md.md)

The prompt carries the policy; the AI writes it into `AGENTS.md` (and links the filename your tool reads to it, if that's a different one), then has a fresh-context Auditor check the file against the prompt. It covers the roles and model tiers (orchestrator/Planner, Executor, Auditor and a cheap Tooler; the AI ranks your tool's models and records the exact name per role), the local `task gate` before every PR, worktrees only through [worktrunk](https://worktrunk.dev) (`wt`), GitHub only through `gh`, the Go testing rules, protected files and the retry cap. It also tells the orchestrator to run each Executor and Auditor as its own agent in a herdr pane, in parallel where tasks are independent. The AI authors the rules; you read them and approve.

> ✅ **Checkpoint:** `AGENTS.md` exists, you've read it, and it covers the roles and rules above. (If symlinks fail, the AI falls back to a copy.)

## 2 Set up your repo with guardrails

**Give it this one prompt:** [`prompts/02-setup-repo.md`](prompts/02-setup-repo.md)

The AI creates your own public repo and, on a feature branch, writes the Go module, a [Taskfile](https://taskfile.dev) with one `gate` task (build, vet, golangci-lint, staticcheck, govulncheck, race tests, a 90% coverage check, each tool at a pinned version), a CI workflow that only runs `task gate` in a job named `guardrails`, and a Dependabot config, installing whatever it needs. Then it opens PR #1 and stops.

**You review and merge PR #1 in the browser.** The Taskfile is the honesty control for everything that follows: from now on the AI needs your sign-off to change it. Before you merge, wait for the `guardrails` check to go green; if it's red, tell the agent to fix it on the same PR. Then move on to step 3.

## 3 Lock the repo with a ruleset

**Give it this one prompt (after merging PR #1):** [`prompts/03-ruleset.md`](prompts/03-ruleset.md)

The AI creates the ruleset. It waits for the merge so the ruleset doesn't get in the way of PR #1. Verify in the browser (repo Settings, Rules):

- a pull request is required before merging
- the `guardrails` check is required
- nobody can bypass
- enforcement is Active

If the ruleset is wrong, fix or delete it in the browser. Dependabot PRs will appear later; ignore them.

## 4 Build the semantic font renderer

**Give it this one prompt:** [`prompts/04-build.md`](prompts/04-build.md)

Two parts, one prompt. First the AI proposes `spec/target.html`: one sentence where each word carries inline styles driven by its meaning, with a short comment explaining the mapping. It opens a small PR and gives you the file path. Open it in a browser (on WSL2, run `explorer.exe .` in that folder); if you like it, **merge it (merging is the approval)**, then tell the agent "merged" and it carries on.

Then it builds the renderer itself: a Planner task list, Executors (in parallel where tasks are independent) that implement it with their own tests (90%+ coverage), `task gate` run by the orchestrator, an Auditor review, looping until its output for the target sentence correctly matches `spec/target.html` (the same relative styling decisions, not byte equality). That match is the stop condition; the gate must also be green to open the PR. You see a PR when it's done, or a question if it's stuck. If it asks because it hit its retry cap, answer with one of: accept the gaps and open the PR, allow one more round, or loosen the match to the few most emphasized words. If you're past minute 75 of the workshop, tell it to use two axes (valence and salience) and three style properties. Out of time? Review whatever PR or question is open; the PR is the deliverable.

Review the PR like any other: the Taskfile, lint config, CI, Dependabot config, `AGENTS.md` and `spec/target.html` should be untouched, and the tests should assert behavior rather than copy the code's output. Run the build and run commands from the PR description, open `spec/out.html` next to `spec/target.html`, and compare. Merge it yourself; the AI never does.

> **If a review bot comments** (Copilot, CodeRabbit or similar; the workshop doesn't need one): read its comments before you merge. The prompts stop at an open PR and the AI doesn't wait for bots, so to act on a comment, tell the agent "Address the open review comments on PR N" and review the result like any other change.

> **Watch:** the AI writes the tests, so the Auditor and you are the check that they say something. A test that passes without the feature working is the failure to look for.

## 5 Optimize (optional stretch goal)

Skip this if you're behind. **Give it this one prompt:** [`prompts/05-optimize.md`](prompts/05-optimize.md)

"Optimize" means both dimensions `testing.B` reports: time **and** allocations. The prompt asks the AI to write the benchmark first; then the orchestrator captures its own baseline and runs `benchstat` after each round, checking the output stays byte-identical; the Executor makes one change per round, and a round that regresses or changes the output is reverted. The Executor never certifies its own improvement. One PR at the end.

The numbers below are illustrative (one presenter's run on Apple-silicon hardware, not a spec); expect the same shape, not these values:

| Round | Change | ns/op | B/op | allocs/op |
|---|---|---|---|---|
| 0 | Map lookups per word, regex tokenizer, a new slice per call | ~380000 | ~210000 | ~4100 |
| 1 | Precomputed dictionary as a slice-backed hash, hand-rolled scanner | ~95000 | ~48000 | ~900 |
| 2 | Single preallocated result slice, reused window buffer | ~61000 | ~9600 | ~2 |

## Wrap-up

- Revoke the GitHub token (and any other token you created for this workshop).
- Undo `gh auth setup-git` if you ran it only for the workshop: `git config --global --remove-section credential.https://github.com` and `git config --global --remove-section credential.https://gist.github.com` (skip this if `gh` was already your credential helper).
- Undo the worktrunk `worktree-path` setting the AI added (`wt config show` shows where).
- The PR is the deliverable, not the transcript.

---

**For helpers:** use a classic token with `repo` and `workflow`: a fine-grained token limited to selected repositories can't cover a repo that doesn't exist yet. `gh --attach` needs gh 2.99.0 or newer.

Companion workshop to *"Use of AI in Industry: The Modern Dark Factory."* Bring your own guardrails.
