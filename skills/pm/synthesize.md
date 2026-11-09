# Synthesize — planning session → docs

Turn a finished planning session into durable documentation. Input: the session conversation in context, or a topic the user points at.

**Audit against the codebase before writing.** Check the actual routers, services, and storage first — a document that contradicts the code is worse than no document.

## 1. Functional conclusions → the target repo's `wiki/product/<topic>.md`
The wiki layout is a convention; follow what the repo already has. In a single repo (no `.sdlc/repos.yml`) with no distinct apps, use `wiki/product/<topic>.md`. In a multi-repo umbrella, a doc about one app goes under `wiki/product/<app>/<topic>.md`; a doc spanning apps or governing the platform (cross-repo coupling, contracts) stays at `wiki/product/<topic>.md`.
One file per significant topic:
```markdown
# <Topic / Feature>
## Problem & Context
## Actors & Goals
## Business / Functional Rules   (numbered)
## Mathematical / Analytical Basis   (cite wiki/sources/ if used)
## Decisions   (link dated FDRs)
## Open Questions
```
Significant business decisions get a dated FDR at `wiki/product/fdr/YYYY-MM-DD-<slug>.md` (Context / Decision / Consequences) — with per-app apps (multi-repo), `wiki/product/<app>/fdr/` for a single-app FDR, `wiki/product/fdr/` otherwise.

## 2. Technical decisions → the target repo's `wiki/architecture/`
Update the affected `wiki/architecture/code/*.md` files (create them if the repo has none), keeping any existing template sections — especially `Current State` and `Open Technical Decisions`. Significant architecture choices get a dated ADR in `wiki/architecture/adr/`. Link to the repo's own `specs/*.md` (root-level, if the repo has one) where a decision has normative consequences, rather than restating their content — `specs/` is `/implement`'s territory, not yours.

## 3. Stop
Report what you wrote and **stop for the user to review.** Issues are outward state; they get created only once the user has read these docs and said go.

On that confirmation, read `issues.md` beside this file and follow it.
