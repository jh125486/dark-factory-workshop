# Step 4 — the one prompt the human gives. You review two PRs: the target, then the renderer.

```
Per AGENTS.md, you are the orchestrator. Goal: build a semantic font
renderer in Go. A scoring library, lexiscore, scores each word of a
text on four axes: valence (negative to positive), salience (how
important), surprise (how unexpected) and certainty (hedged to
confident). A renderer turns text into one self-contained HTML page
with embedded CSS where each word's style (weight, size, colour,
slant, spacing, opacity) follows its scores. HTML and CSS because it's
the easiest and most platform-independent output. A thin CLI reads
text from stdin or a file and writes the HTML to stdout, so the demo
runs.

Part 1, the target. First, on a feature worktree, create spec/target.html:
one short sentence where every word is a span with
inline styles driven by its meaning, plus a short comment at the top
explaining the mapping. No JavaScript, no external files. Open a small
PR, tell me the absolute path of the file so I can open it in a
browser, and stop. I'll merge it if I approve it (merging is the
approval). Wait until I tell you it is merged.

Part 2, the renderer. Once spec/target.html is merged and main is updated,
do the rest yourself: Planner task list, a mid-tier Executor that
implements it with its own tests (at least 90% total coverage),
`task gate` run by you, an Auditor review. Iterate under AGENTS.md's retry
cap until the renderer's output for the spec/target.html sentence correctly
matches the file (you judge this by running the CLI on that
sentence and comparing; the Auditor checks it too): the same relative
styling decisions (words it
emphasizes, softens or recolours come out that way too; not byte
equality). That match is the stop condition; `task gate` must also be
green before you open the PR. Score text generally: no special-casing that
sentence. Don't fetch a word list from outside; build any lexicon by
hand. Replace the placeholder package from setup (deleting it and its test is
expected). Don't change protected files.

The PR description has gate.log's
result and the commands to build and run the CLI with the HTML
redirected to spec/out.html (it is gitignored): one form that works
from the worktree path, one for the lab root after merge (after a `git pull` there).
```
