# Step 1 — the one prompt the human gives. The prompt carries the policy; the AI writes AGENTS.md and the human reads and approves it.

```
Write AGENTS.md here: concise bullets, only what your context needs,
for a dark factory where I give a goal once and you deliver a reviewed
PR.
- Roles. Look up your tool's official model list and prices online; use
  one provider, newest generation only, and rank by price rather than
  answering "unclear". You are the orchestrator and Planner (high
  reasoning, but not the priciest model): plan, never code. Executor: the best coding model
  that isn't the priciest. Auditor: the most capable model at high
  effort, fresh context; ACCEPT or REJECT, never fixes. REJECT if tests
  echo the code, anything is weakened or skipped, a test is deleted, a
  protected file changed, a dependency is unflagged, or any rule here or
  in the goal is broken; also try to break the build with inputs the
  tests lack. Tooler: the cheapest model, for setup and CI, on its own
  (no Executor or Auditor). Don't reuse a model across roles unless
  there's no alternative; record the exact name and price per role, and
  ask me only on a tie. Subagents
  see only their task, never spawn subagents, and independent tasks run
  in parallel, each in its own herdr pane; wait with herdr's blocking
  wait, never poll in a loop.
- `task gate` must pass locally before any PR; the PR shows its result
  and a per-file coverage table (no custom tooling).
- Worktrees only via `wt`, never the primary checkout: learn it from its
  help and record the exact create and remove commands in AGENTS.md (no
  hooks). Start every PR branch from freshly fetched `main`. GitHub only
  via `gh`; never push to main; the human merges.
- Go tests: one table-driven `t.Parallel()` test per exported function,
  one blackbox test file per source file, asserting the goal's behavior,
  not the code's output, committed before the code they test. Keep the
  CLI `main` thin; 90% coverage; never weaken a test. Guardrails and
  these rules hold unless I explicitly allow an exception.
- Protected, my sign-off needed: Taskfile, lint config, CI, Dependabot,
  AGENTS.md, spec/target.html, repo settings and rulesets.
- Same failure after two rounds: stop and escalate.
- If your tool reads another instructions filename, symlink it to
  AGENTS.md (a copy if symlinks fail).

Then have the Auditor check AGENTS.md against this prompt for rules that
are missing, contradicted or invented. Fix what it finds and tell me
what changed.
```
