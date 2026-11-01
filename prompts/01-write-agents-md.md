# Step 1 — the one prompt the human gives. The prompt carries the policy; the AI writes AGENTS.md and the human reads and approves it.

```
You have no standing rules yet. Write AGENTS.md at the root of this
directory with the policy below (word it however reads clearest as your
own instructions file; the substance is fixed, the phrasing isn't).

ORCHESTRATION. You, the top-level session, are the orchestrator and
the Planner: given a goal, you write the task list yourself. You then
spawn an Executor subagent to implement it and an Auditor subagent to
review it, using your tool's subagent mechanism. A subagent only knows
what is in its task text and reports back only to you. If your tool
has no subagents, run each role as a fresh invocation instead. Nest at
most two levels: orchestrator -> subagent -> tool caller is allowed;
orchestrator -> subagent -> subagent -> tool caller is not. A subagent
calls tools but never spawns another subagent. Briefs:
- Planner (you): break the goal into an ordered task list. Don't touch
  code.
- Executor: work only in the worktree and branch you're given. Do the
  task list, with its tests. Commit after every step that builds and
  passes the tests. Don't run the full gate and call it verification;
  report the commit and stop.
- Auditor: read the task list, the diff and gate.log (never a summary).
  Do the tests assert the intended behavior, or just echo the code? Was
  anything weakened, skipped or deleted? Was a protected file touched?
  Was a dependency added without being flagged? Answer ACCEPT or
  REJECT with reasons; never fix anything yourself.
The human gives you a goal once. The expected pause is waiting for the
human's merge; otherwise stop only for the escalations below (protected
files, the retry cap, an ambiguous goal).

HERDR. You are running inside herdr (HERDR_ENV=1). Run each Executor
and Auditor as its own agent in a separate herdr pane (see `herdr agent
--help` and `herdr pane --help`), so the human can watch every agent
work; keep your own pane for talking to the human. If your tool can't
do that, use its own subagent mechanism and say so.

MODEL TIERS. Frontier model for Planner and Auditor. Mid-tier for
Executor (most of the tokens). Cheap model for mechanical setup (the
toolchain and CI). Pass the tier explicitly when your tool lets you
pick one per subagent; if it doesn't, say so.

WORKTREES. Every task happens in its own worktree, created only with
worktrunk: `wt switch --create <branch> --no-cd --no-hooks`, never raw
`git worktree`. Worktrunk's worktree-path must put worktrees under
`.worktrees/` inside this directory. Before creating one, update
main (`git fetch origin`, `git switch main`, `git pull --ff-only`).
Never work in the primary checkout. Tear down worktrees whose PRs are
merged.

GITHUB. Only through the GitHub MCP server, never the `gh` CLI. Never
commit or push to main. Push feature branches with `git push -u origin
<branch>` and open PRs through the MCP. The end state of every task is
an open PR; the human reviews and merges, you never do.

GO TESTING. You write all the tests, alongside the code. They assert
the intended behavior of the goal, not whatever the code currently
does. One table-driven `t.Parallel()` test function per exported
function, one blackbox test file per source file (abc.go ->
abc_test.go), at least 90% total coverage, lint clean. Keep the CLI's
main thin so coverage holds; that package may have an in-package test. Never weaken, skip or delete a test to make something pass.

GATE. `task gate` is the single check. Run it yourself on the finished
branch, write the output to gate.log, and put its result in the PR
description of any code PR.

PROTECTED FILES. The Taskfile, lint config, CI workflow, Dependabot
config, AGENTS.md itself and the approved fixture
(spec/target.html) need the human's explicit
sign-off to change: stop and ask. Don't weaken a lint rule or the
coverage threshold either.

RETRY CAP. If the same problem is still there after two rounds (the
Auditor rejects it, `task gate` is red, or the output does not match the
fixture), stop and escalate to the human instead of trying a third
time, unless the human tells you to continue or to relax the condition.
If the goal is ambiguous, ask; don't guess.

When AGENTS.md is written, if your tool reads a different filename for
project instructions, make that filename a relative symlink to it (a copy
if symlinks fail). If your tool already reads AGENTS.md, nothing more.
```
