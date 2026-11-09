---
name: prototype
description: >
  Build throwaway code that answers a design question — whether a state model, data shape, or
  layout is right — then keep the answer and delete the code. Use when a question can't be
  settled on paper, or a pm plan work item's next step is a prototype.
---

# Prototype

A prototype is **throwaway code that answers a question**. The question decides the shape, and the answer is the only thing that survives.

## The worktree

A prototype lives in its **own worktree**, like any other development, and is discarded when it's no longer needed. It never merges to `main`.

Read `.sdlc/sdlc-config.yml` at the repo root for `prototype.{worktree_path,branch}` and `default_branch` — if that file is missing, stop and tell the user the target repo isn't configured for this skill (run the `setup` skill) rather than guessing a worktree convention. Then:
`prototype.worktree_path` is relative to the repo root, and the worktree directory must be gitignored: check with `git check-ignore <dir>`; if it isn't ignored, tell the user and ask before adding it to `.gitignore`.
```bash
git worktree add <prototype.worktree_path> -b <prototype.branch> <default_branch>
cd <prototype.worktree_path>
```
Run whatever dependency-install step the repo's `.sdlc/tooling.md` documents (e.g. `uv sync --extra dev`) before running anything — a fresh worktree has no installed dependencies yet.
Where the prototype needs the repo's layout or gate commands, read `.sdlc/repo-structure.md` and `.sdlc/quality-gates.md` (both optional here; if missing, discover from `CLAUDE.md` or `AGENTS.md` and `README`).
Everything the prototype builds stays on that branch. Because nothing there is bound for `main`, the repo's conventions don't apply inside it — that freedom is the point.

**What survives the worktree** is only ever an answer, never the code that produced it:
- **The verdict** → `/pm`, which records it in the work item's detail file.
- **A validated design** — the chosen UI option, the confirmed state model — → a dated decision record in the target repo's `wiki/architecture/adr/`, through `/pm`, so the decision outlives the branch. Not `specs/`: that stays tied to shipped code and is `/implement`'s territory, updated when the real version is actually built.
- **Real code** is written later, from an issue, through `/implement`. Prototype code is never promoted; it was built under prototype constraints and it stays behind.

Keep the worktree while it's still teaching you something. Once it isn't, `git worktree remove <prototype.worktree_path>` and let the branch go.

## Pick a branch

Identify the question — from the user's prompt, the surrounding code, or by asking if the user is around:

- **"Does this logic, state model, or data shape feel right?"** → [LOGIC.md](LOGIC.md). A tiny interactive terminal app that pushes the model through cases that are hard to reason about on paper.
- **"What should this look like?"** → [UI.md](UI.md). Several radically different variations on one route, switchable from a floating bar.

The branches produce very different artifacts, so a wrong pick wastes the whole prototype. If the question is genuinely ambiguous and the user isn't reachable, default by where the code lives — a service, model, or storage question → logic; a page or component → UI — and state the assumption at the top of the prototype.

## Rules for both

1. **One command to run.** A single command from the worktree, e.g. a script or a `Makefile` target. The user starts it without thinking.
2. **No persistence, and never the real store.** State lives in memory. Where the question genuinely needs stored data, read from a **copy** — whatever read-only backup or scratch data source the target repo's `CLAUDE.md`/`AGENTS.md`/`specs/` names — and never write anywhere into the repo's real data store. The worktree is throwaway; the data isn't.
3. **Skip the polish.** No tests, no error handling past what makes it runnable, no abstractions. Learn fast; the branch is going away.
4. **Surface the state.** After every action, or on every variant switch, show the full relevant state so the change is visible.

## When done

Capture the **answer** and the question it answered, and hand both to `/pm`. If the user is around, that's a quick conversation about what the session taught them; if not, leave it in a `NOTES.md` on the branch to be filled in before the worktree goes.

A design the answer settles — the winning UI variant, the validated state model — goes to the target repo's `wiki/architecture/adr/` as a dated decision record, via `/pm`, in the same pass. That's what makes the prototype disposable: once the decision is written down somewhere permanent, the code that produced it has no reason to survive.
