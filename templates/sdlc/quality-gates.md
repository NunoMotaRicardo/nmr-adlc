# Quality gates — <repo>

<!-- Prose companion to `quality_gates.*` in sdlc-config.yml. The commands themselves live ONLY there.
     Read by /pr-review (to know what CI already covers) and the reviewer (e.g. where SAST runs). -->

## Gates

- Commands: see `quality_gates.*` in `sdlc-config.yml` (test, lint, format_check, type_check).
- CI: `<.github/workflows/ci.yml>` runs <the same commands>. Local gates and CI must not disagree.
- Security scanning (SAST/dependency audit): <tool, where it runs — skills leave it to this gate>.

## Merge requirements

<!-- Required checks, review count, branch protection, coverage threshold, decision-capture rules, etc. -->

## Repos without code gates

<!-- If the repo is docs-only and `quality_gates` is empty, say so and name what stands in for a gate
     (e.g. a link checker for wiki/). Delete this section otherwise. -->
