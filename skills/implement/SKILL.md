---
name: implement
description: >
  Run the implementation loop for one GitHub issue — or a batch of them in one
  worktree and one PR: feasibility gate, a coder subagent in a worktree, a reviewer subagent
  (max 3 rounds), a PR that closes the issue(s), then the feedback loop that drives every
  review comment on that PR to resolved. Use when the user fires "/implement <issue#>",
  "implement issue N", "implement issues N to M", "build issue N", or asks to handle the
  automated or human review comments on a PR. Requires gh auth and a planned issue.
---

# Implement

You are the **orchestrator**, in the main thread. You drive one GitHub issue to a merge-ready PR by delegating to two subagent roles (a fresh-context subagent each where your tool supports it; otherwise separate inline phases). You delegate, gate, and report — the code and tests are theirs to write.

Folder names: `<specs>`, `<pm_docs>` and `<user_docs>` are the repo's folders from the **Folders** table in `.sdlc/repo-structure.md` (defaults `specs`, `docs`, `wiki`; a value may be `none`). If that table is missing, use `specs`, `wiki`, `none` and suggest running the `setup` skill.

**The briefs** live beside this file: `coder.md` and `reviewer.md` for the subagents (you pass a subagent the absolute path and tell it to follow that file — you never read them yourself), and `pr-feedback.md`, which you read yourself at step 8.

**More than one issue in the input** — a range, a list, or a set description — is **batch mode**. Read `batch.md` beside this file and follow it: it plans the set, resolves the dependency order, and runs the loop below once per issue in one shared worktree, ending in one PR. The rest of this file still binds; `batch.md` only says what differs.

## Configuration
Every repo carries its own `.sdlc/sdlc-config.yml`, at the root `git rev-parse --show-toplevel` reports. Read it first and take every value below from it — the identifiers differ per repo, and a hardcoded one silently targets the wrong project. If the file is missing, **stop** and tell the user the target repo isn't configured for this loop (run the `setup` skill) rather than guessing or falling back to another repo's values:
- `github.{owner,repository,slug}` — every `gh` call; `github.project.{number,id,status_field_id}`, `github.project.statuses.*` — board moves (optional; see below).
- `default_branch` — what worktrees branch from and PRs target, written `<default_branch>` below.
- `implement.{worktree_path,branch}` — the path and branch-name templates, `{issue}` and `{slug}` substituted. `worktree_path` is relative to the repo root.
- `implement.max_changed_loc` — the feasibility gate's size ceiling.
- `quality_gates.*` — the commands the subagents run.

Board move, used verbatim at steps 2 and 7 with a different status each time:
```bash
item_id=$(gh project item-add <project.number> --owner <github.owner> \
  --url https://github.com/<github.slug>/issues/<NNN> --format json --jq '.id')
gh project item-edit --id "$item_id" \
  --project-id <github.project.id> \
  --field-id <github.project.status_field_id> \
  --single-select-option-id <github.project.statuses.<STATUS>>
```
(`item-add` is idempotent. Re-read the config if a call 404s — recreating the project changes the IDs.)

**Token scope errors.** `gh project` calls need `read:project` + `write:project` scopes. If a call fails with `missing required scopes`, stop the board move, tell the user which scopes are missing and to run `gh auth refresh -s read:project -s write:project`, then retry once they confirm. Never read or export tokens from files.

**Board moves are optional.** If `github.project` is absent from `.sdlc/sdlc-config.yml`, skip every board move in this loop and continue.

## Inputs
An issue number (from `$ARGUMENTS`, e.g. `/implement 7`); if missing, list open issues (`gh issue list`) and ask which one. Requires `gh` authenticated, the issue present with the standard template (from `/pm`), repo root, clean tree.

**Several issue numbers** — `#139 to #144`, `#139, #141, #144`, or a named set — go to `batch.md` first, then back here per issue.

**Entering at step 8.** A PR number instead of an issue number — or a request to handle the review comments on an open PR — skips straight to step 8's feedback loop. Resolve the issue number from the PR body's `Closes #<NNN>` for the board move.

## Loop

### 1. Feasibility & scoping gate
Audit the issue yourself against the current codebase — read the issue, the specs it references, and the affected code (its location is documented in the target repo's own `.sdlc/repo-structure.md`; read that first if unsure). Is it one vertical slice within `implement.max_changed_loc` including tests? Are the Acceptance Criteria testable? Are the referenced `<specs>/` files — the repo's normative contracts and invariants, located per `.sdlc/repo-structure.md` — complete and current? If the repo has no `<specs>/` directory at all, say so and proceed on the issue text and code alone; don't invent contracts that don't exist.

**Feasible** → continue. **Too big or under-specified** → propose a split (or a `/pm` re-slice), tell the user, and **stop**. Run this gate on every issue, including the ones that look small.

### 2. Read, confirm, start
- `gh issue view <NNN>` — Summary, Vertical Slice, Legacy Compatibility & Sync, Implementation Tasks, Acceptance Criteria, Depends On, Technical Notes.
- An unclosed issue in `Depends On` → stop and tell the user.
- Confirm the plan with the user before creating a worktree, only if you think there is a risk of misalignment, ambiguity, missing information or too many Lines of Code. If the user confirms, continue; if they do not, **stop**.
- Board move → `in_progress`.

### 3. Worktree
```bash
git worktree add <implement.worktree_path> -b <implement.branch> <default_branch>
```
The worktree directory must be gitignored: check with `git check-ignore <dir>`; if it isn't ignored, tell the user and ask before adding it to `.gitignore`. All subsequent work happens on that path and branch.

### 4. Implement — coder subagent
Delegate to a fresh-context subagent if your tool supports it (e.g. Claude Code's Agent tool, Codex/Copilot/Gemini equivalents); otherwise run the brief inline as a separate phase, re-reading the brief file and not reusing the orchestrator's reasoning. The brief: read `<this skill's directory>/coder.md` and follow it, with the issue number, the worktree path, the relevant spec/interface excerpts, and the quality gates from `sdlc-config.yml` (`quality_gates.*`). Relay its summary — files changed, tests added, gate results. A reported blocker or ambiguity → surface it to the user and stop.

### 5. Review — reviewer subagent (max 3 rounds)
Using the same delegation rule (fresh-context subagent if supported, otherwise a separate inline phase), run the reviewer: read `<this skill's directory>/reviewer.md` and follow it, for issue #NNN, round R, in the worktree. It returns a verdict, findings, and a PR description draft.
- **CHANGES REQUESTED or a failing gate**, rounds 1–2 → relay the blocking findings, re-spawn the coder subagent with them (same worktree and branch), re-review at round + 1.
- **Round 3 still failing** → **stop and escalate**: report branch, issue number, and findings. No push, no PR.
- **APPROVED** → continue, carrying NON-BLOCKING notes into the PR body.

### 6. Check off the issue's Implementation Tasks
Closing an issue from a PR does not tick plain checklist lines — only native sub-issue links do that (see `/pm`'s `issues.md`). Do it explicitly, before opening the PR:
- `gh issue view <NNN> --json body -q .body` for the current body.
- Compare what was actually built (the coder's summary, plus any fix rounds) against each `- [ ]` line under **Implementation Tasks**, and flip the completed ones to `- [x]`. Check only what is genuinely done — anything descoped or deferred stays unchecked, with the reason noted in the PR body.
- Write the whole body back — `gh issue edit <NNN> --body "<full updated body>"` — there is no partial-edit flag. A short inline Python snippet does the `- [ ]` → `- [x]` swap on the exact matched lines, where a broad find/replace would hit unrelated text.
- Verify with a second `gh issue view <NNN> --json body -q .body` that the boxes landed.

### 7. PR (do not merge)
- Commit in the worktree (repo commit conventions, Claude trailer), push, and open the PR with the reviewer's draft body:
  ```bash
  gh pr create --base <default_branch> --head <implement.branch> --title "<title>" \
    --body "<body incl. 'Closes #<NNN>', quality-gate overview, Legacy Coexistence status>"
  ```
- Board move → `in_review`, once, right as the PR opens.
- Tell the user the PR is open and stop. Reviews now happen on GitHub — any automated reviewers configured on the repo, human reviewers, or a `/pr-review` run. Never auto-merge.

### 8. PR feedback loop (only when the PR draws review comments)
Read `pr-feedback.md` beside this file and follow it: triage every open review comment, delegate the confirmed fixes to the coder subagent, then reply to and resolve every thread. This step owns the board while the PR is under review — triage is the first point that knows which findings are real, so it decides whether the issue drops back to `in_progress` or stays at `in_review`.

A PR nobody comments on skips this step entirely.

### 9. After the user merges
- `git worktree remove <implement.worktree_path> 2>/dev/null || true` (idempotent — `/pm`'s post-merge duty removes it too, so either order is safe).
- Hand off to `/pm` post-merge duty: update the affected `<pm_docs>/architecture/` descriptive docs' Current State, close the issue if the PR didn't, remove the worktree, board → Done, unblock dependents.
- Report a one-paragraph summary, including any issues just moved to Ready.

## Rules
- One issue per run, unless `batch.md` is driving — then one issue at a time, in dependency order, on one shared branch. Respect `Depends On` either way.
- Code, tests, specs and user docs are the subagents' work — delegate.
- The review gate always runs; a PR opens only on a green suite and green gates.
- Reference cross-repo issues using `owner/repo#N` format (bare `#N` remains legal within the local target repository).
- Never merge. Stop on any unresolved blocker and report clearly.
