# Step 3 — the one prompt the human gives, only AFTER merging PR #1 in the browser.

```
PR #1 is merged. With the GitHub MCP, create a ruleset on the default
branch that requires a pull request (0 required approvals, since I'm
the author), requires the `guardrails` status check, blocks force pushes and
deletions, allows no bypass, and has enforcement active. Read it back to me as a short table (rules, approvals, required
check, bypass list). Then update local main (`git fetch origin`,
`git switch main`, `git pull --ff-only`) and tear down the merged
worktree.
```
