# Step 5 (optional stretch goal) — the one prompt the human gives. Ends with one PR you review and merge.

```
Per AGENTS.md, you are the orchestrator. Goal: optimize the renderer for
both time (ns/op) and allocations (allocs/op), with the existing tests
as the safety net. Never weaken, skip or delete a test.

Update main and tear down any merged worktrees, then work on ONE branch made with `wt`. First, have an
Executor write a benchmark for the renderer's hot path (rendering a
realistic paragraph of text end to end) as the first commit. Before changing anything, capture the baseline yourself:
    go test -bench=. -benchmem -count=10 -run='^$' ./... | tee bench_before.txt

Then, a round at a time, spawn a mid-tier Executor with ONE specific
change. Look at the real code first.
After each round YOU run the benchmark again into bench_round<N>.txt,
compare with `benchstat <last kept round's file> bench_round<N>.txt`
(bench_before.txt for round 1), and run the tests. Each round is one commit. If a round regresses or breaks a test, `git revert` its commit and
try a different change; two reverted rounds in a row means stop and
open the PR with what is already good (this step's own stop rule; AGENTS.md's retry cap applies otherwise). Stop when allocs/op stop dropping (about three
rounds), run `task gate`, have an Auditor review the final diff, and open ONE PR with gate.log's result and
every round's before/after ns/op, B/op and allocs/op. I'll review and
merge.
```
