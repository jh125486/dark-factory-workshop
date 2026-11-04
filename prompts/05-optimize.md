# Step 5 (optional stretch goal) — the one prompt the human gives. Ends with one PR you review and merge.

```
Goal: make the renderer faster and allocate less.
First commit a benchmark of its hot path (a realistic paragraph, end to end) and record a baseline.
Then one change per round, each its own commit: you, not the Executor, benchmark it against the last kept round with benchstat and confirm the output is byte-identical.
Revert any round that regresses or changes the output; stop after two reverts in a row or when allocs/op stop dropping.
Open one PR listing each round's ns/op, B/op and allocs/op.
```
