---
name: diagnose
description: >
  Find the cause of a defect or anomaly in the codebase or the stored data — wrong values,
  a failing path, drift between two representations of the same data. Use when something is
  wrong and the mechanism isn't known, or a pm plan work item's next step is a diagnosis.
---

# Diagnose

`.sdlc/repo-structure.md` is required for code repos: if it is missing, say the repo isn't fully configured (run the `setup` skill) and continue with a stated fallback: read `CLAUDE.md` or `AGENTS.md`, `README` and the tree to discover the layout, flagging the loss of accuracy in your report. `.sdlc/tooling.md` is optional: if missing, discover the run/test commands the same way.

Find the **mechanism** — the specific code path or data condition that produces the wrong result — and prove it. A diagnosis that names a suspect without demonstrating it is a hypothesis wearing a verdict's clothes.

You investigate; you don't fix. What you return goes to `/pm`, which records it in the work item's detail file. A confirmed defect ready to be built becomes a work item whose next step is `write-document`.

## Scope

**Yours**: the target repo's application code (its layout is documented in that repo's own `.sdlc/repo-structure.md` — read it first, don't assume a layout from another repo in this platform), the tests, and the data at rest, whose normative shape and invariants are documented in that repo's `specs/*.md` (root-level, code-bearing repos only — if absent, treat the code and any in-repo reference docs as the only ground truth). Includes drift between any two representations the repo maintains of the same data (e.g. a current store and a retired or legacy one it coexists with). Local, reproducible, read-only investigation.

**Out of scope**: live production operations — running-system logs, connectivity, schedule behavior, a job that failed last night. When the evidence you need is only in the running system, say so and hand it back to the user rather than reasoning about production from here.

Data is read-only throughout. Reading the store is diagnosis; writing to it is a fix, and fixes go through `/implement`.

## Method

### 1. State the symptom precisely
What is observed, where, and what was expected instead — with the actual values. "The numbers look wrong" is not a symptom; "`<record>` reads `0` in the store where the upstream source of truth reports `94.9B`" is.

### 2. Reproduce it
Reproduce the symptom locally before theorising: a failing test, a direct read against the store (using whatever query/scan mechanism the repo's `.sdlc/tooling.md` or `specs/` documents), or a direct call to the service. A symptom you can't reproduce is the first finding — report what you tried and what the store actually shows.

The reproduction is what makes the rest binary. With it, every hypothesis becomes checkable; without it, the investigation runs on plausibility.

### 3. Narrow
Bisect the path between correct input and wrong output, through this repo's own layers — read `.sdlc/repo-structure.md` or `specs/layering-rules.md`-equivalent for what those are here (e.g. provider → domain model → storage write → storage read → service → API is one repo's shape, not a universal one). Check the value at each boundary and find the first one where it's already wrong. Where the defect appeared in time, `git log`/`git bisect` on the touching files narrows the same way.

Follow the evidence rather than the most likely story. The suspicious-looking function is a hypothesis like any other, and gets checked, not assumed.

### 4. Confirm the mechanism
Name the exact line or data condition, and demonstrate it: change that one thing and watch the symptom move. A cause that can't be made to appear and disappear on demand is still a suspect.

### 5. Report
- **Symptom** — observed vs. expected, with real values.
- **Reproduction** — the exact command, test, or query, so anyone can re-run it.
- **Mechanism** — the file, line, or data condition, and the demonstration that it's causal.
- **Blast radius** — what else this touches: other symbols, other date ranges, other datasets, the legacy CSV, downstream dashboards. Check it; don't assume the report's example is the only case.
- **Fix sketch** — what a fix would change and what regression test would pin it. Enough for `/pm` to slice an issue, not the fix itself.
- **What stayed unexplained** — anything the evidence didn't cover.
