# Step 4 — the one prompt the human gives. You review two PRs: the target, then the renderer.

```
Goal: a semantic font renderer in Go. A library, lexiscore, scores each
word on valence from the VADER lexicon (MIT; commit it with attribution)
and derives salience, surprise and certainty (hedged to confident). A
renderer turns text into one self-contained HTML page whose CSS sets
each word's weight, size, color, slant, spacing and opacity from its
scores, using appropriate a11y techniques. A thin CLI reads stdin or a
file and writes HTML to stdout.

First, open a PR with spec/target.html: one short sentence, each word a
span styled by its meaning, a comment explaining the mapping, no JS or
external files. Give me its path; merging it is my approval.

Then build the renderer until its output for that sentence makes the
same relative styling choices as the target (not byte-equal). Score text
generally, no special-casing. The PR has build/run commands that write
spec/out.html (gitignored) and, if Playwright is installed, screenshots
of the target and your output.
```
