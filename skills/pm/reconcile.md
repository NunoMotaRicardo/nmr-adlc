# Reconcile — post-merge state and backlog

Two entry points that interlock: the post-merge pass runs the triage as its last act.

## Post-merge (called from `/implement` step 9, or on demand)

After an issue's PR merges:
1. `git diff` review of what landed.
2. Update the `Current State` sections of the affected `wiki/architecture/code/*.md` descriptive docs. The repo's normative `specs/*.md` (including `specs/contracts/` if the repo has them) are `/implement`'s own territory, updated in the same PR as the code — verify that happened rather than duplicating it here.
3. Close the issue if the PR didn't: `gh issue close <NNN>`.
4. Remove the issue's worktree (`implement.worktree_path`, relative to the repo root) and prune stale metadata:
   ```bash
   git worktree remove <implement.worktree_path> 2>/dev/null || true
   git worktree prune
   ```
5. Board move → `done` (skipped if no `github.project` is configured).
6. Run the triage below to unblock dependents.
7. Report a one-paragraph summary, including any issues just moved to Ready.

**Cross-repo contracts and system-overview docs have no single issue owner — check them explicitly.** `/implement` naturally updates the *specs* an issue targets in that repo (including any `specs/contracts/*.md` the change produces or consumes), but system-overview architecture docs (e.g. a `wiki/architecture/code/` overview doc — check `wiki/_Sidebar.md` for its current name; in multi-repo mode, the umbrella's cross-system one) describe the shape *across* modules or repositories, so they drift silently. On every merge, ask: did this change a contract in `specs/contracts/`, or a layer's **public surface** — a new service, model, router/entry surface, or provider/storage/core util, or a new or renamed module? If so, check whether the overview doc's `Current State` needs the same update in this same pass. Flip planned→shipped markers, and delete "planned (#N)" notes once #N has merged.

## Backlog triage (on demand, or step 6 above)

1. Find every open issue whose `Depends On` references recently closed issues:
   ```bash
   gh issue list --search "#<NNN> in:body" --state open --json number,title,body
   ```
   For each, read its full `Depends On` line and check every listed issue's state (`gh issue view <dep#> --json state -q .state`):
   - **All dependencies closed** → board move → `ready` (without a board: just report "unblocked"). Report "unblocked".
   - **Any still open** → leave at Backlog, naming which ones still block it.

   Every dependent the search returned is accounted for — moved, or explicitly left with a reason.
2. Report the full newly-Ready list with one-line summaries.

## "What should I work on next"
Rank the Ready issues by: **unblocked** (no open `Depends On`) first, then **milestone alignment** (earlier phase first), then **estimated size** (under `implement.max_changed_loc` preferred). Present the ranking with a one-line rationale per pick, and ask the user to confirm or override.
