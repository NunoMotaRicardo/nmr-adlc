# Slice — specs → GitHub issues

Cut documented work into buildable issues. Requires the user's explicit go-ahead: issues are outward state.

## Parent issue types (fixed vocabulary)
Before creating a new parent, scan open issues for one whose scope already covers the work and re-use it if it fits.

| Type | When to use | Label |
|---|---|---|
| **Epic** | A work stream with an indeterminate timeline; an ongoing capability area | `epic` |
| **Feature** | Time-bounded development of a new, self-contained capability | `feature` |
| **Task** | A purely operational or maintenance container (runbook, chore batch) | `task` |

Parent + sub-issue structure is a **recommendation**. Use it where it adds clarity; a flat set of issues with no meaningful grouping is fine as it is.

## 1. Read and audit
Read the relevant `<specs>/` (target repo, if present) and `<pm_docs>/product/` files, then audit the codebase for what already exists. A repo with no `<specs>/` directory at all (e.g. one with no implementation loop) isn't an error — proceed on `<pm_docs>/` and the code alone and say so in the issue's Technical Notes.

## 2. Cut vertical slices

Read the target repo's `.sdlc/repo-structure.md` for its actual layers — a common shape is Domain → Infrastructure → Service → API, but never assume that's this repo's shape; use whatever it documents. If that file is missing, stop and tell the user the target repo isn't configured for `/pm` slicing (run the `setup` skill) rather than guessing a layer shape. Each issue spans however many of those layers one narrow behavior actually touches. LOC Check: inspect the target files in the repo's application-code root (per `.sdlc/repo-structure.md`) and its test suite. Estimate `(Added + Modified Code Lines) + Test Lines`. If `Total > implement.max_changed_loc`, the issue MUST be split into multiple dependent issues before proceeding.

### Slicing Strategies for Oversized Features
When a complete feature slice exceeds `implement.max_changed_loc` (check the target repo's `.sdlc/sdlc-config.yml` for the actual number), split it into smaller sequential issues using one of these strategies and link them with `Depends On`. Substitute the repo's own layer/module names for the generic ones below (`.sdlc/repo-structure.md` names them).

    * **Strategy 1: Data-First Split (Preferred for storage & schema features)**
      * **Issue 1 (Data Layer)**: Domain schemas/models + storage repository + repository unit tests (~half the LOC ceiling).
      * **Issue 2 (Service & API Layer)**: Service business logic + API router + route integration tests (~half the LOC ceiling, `Depends On: #Issue1`).

    * **Strategy 2: Capability / Scenario Split (Preferred for complex workflows)**
      * **Issue 1 (Core Happy Path)**: Primary read/write flow + standard validation + base tests (~60-70% of the LOC ceiling).
      * **Issue 2 (Advanced Logic & Reversals)**: Reversal/rollback logic, bulk options, or edge cases + focused tests (~half the LOC ceiling, `Depends On: #Issue1`).

    * **Strategy 3: Interface & Provider Split (Preferred for external API integrations)**
      * **Issue 1 (DTOs & Client Contract)**: Data contracts, abstract provider interface, and mock fixtures (~40% of the LOC ceiling).
      * **Issue 2 (Provider Implementation)**: Live external provider implementation + integration tests (~60% of the LOC ceiling, `Depends On: #Issue1`).

## 3. Create the issues
`gh issue create --repo <target.slug>` in the resolved target repository (the current repo in single-repo mode, else via `repos.yml`; see `SKILL.md` Configuration & Issue Routing) on the right **phase milestone**, with this template as a recommendation — adapt or omit sections that don't apply:
```markdown
## Summary
## Vertical Slice        (which layers this touches and what gets built in each — per .sdlc/repo-structure.md)
## Coexistence & Sync     (any retired/reference code area this must stay compatible with, and how — or "N/A" if this repo has none)
## Implementation Tasks  (checklist)
## Acceptance Criteria   (testable behaviors, not steps)
## Technical Notes       (spec/interface links, contracts, gotchas)
## Estimated LOC
    - one line per layer this issue touches, LOC per layer
    - Unit & Integration Tests: LOC
    - **Total Estimated LOC**: at or under `implement.max_changed_loc` from this repo's `.sdlc/sdlc-config.yml`
## Depends On            (#issue refs, or "none")
```

## 4. Label
Phase label (e.g. `phase-3-financials`) + a component label naming the part of the repo this touches (check `gh label list --repo <target.slug>` for what's already in use in that repo, rather than inventing a new one). Parent issues get their type label (`epic`, `feature`, `task`).

## 5. Link children to their parent
A child issue under a parent gets a **native GitHub sub-issue link**, using `github.slug` of the target repository for the `<owner>/<repo>` segments:
```bash
child_id=$(gh api repos/<owner>/<repo>/issues/<child_number> --jq '.id')
gh api repos/<owner>/<repo>/issues/<parent_number>/sub_issues -X POST -F sub_issue_id=$child_id
```
Note `-F` (not `-f`) — `sub_issue_id` must go as an integer, and it's the issue's internal `id`, not its `number`. Update the parent's own checklist body to reference the new issues as well; the sub-issue link and the checklist line are both expected, neither substitutes for the other. Verify with `gh api repos/<owner>/<repo>/issues/<parent_number>/sub_issues --jq '.[].number'` and confirm the new numbers appear.

## 6. Set each new issue's board Status
If `github.project` is not configured, skip this step. Otherwise new issues auto-add to the project at **Backlog**, so this stays a manual step. Use the board move from `SKILL.md` with the target-repo issue URL:
- `Depends On: none` **and** not a parent → **Ready**.
- Any unresolved `Depends On`, or a parent (epic/feature/task) → leave at **Backlog**. Parents aren't directly buildable, so they never reach Ready whatever their own `Depends On` says.

## 7. Report
Every created issue number with a one-line summary, the sub-issue link confirmed for each, and each Status confirmed set (when a board is configured).
