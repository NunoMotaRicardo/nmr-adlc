# Repo structure — <repo>

<!-- Read by /implement (coder, feasibility gate), /pm (issue slicing), /diagnose, /pr-review and the
     reviewer. They never assume a layout: this file is where they learn it. Replace every <...>.
     If this file is missing, /pm and /implement stop and say the repo isn't configured. -->

## Layers

<!-- The layers of the application, top-down, and the rule for which may depend on which.
     /pm slices each issue vertically across these layers. Example shape: API -> Service -> Domain -> Infrastructure. -->

| Layer | Path | Responsibility |
|---|---|---|
| <layer> | `<path/>` | <one line> |

Dependency rule: <e.g. "each layer imports only from the layer below">.

## Where things live

- Application code: `<path/>`
- Tests: `<path/>` (how they mirror the source tree, naming convention)
- Specs / contracts (normative, if any): `<specs/>` — written by /implement in the same PR as the code. Say "none" if the repo has no `specs/`.
- Documentation: `wiki/` (owned by /pm)
- Data at rest (if any), and how to query it: <path or mechanism>

## Entry points

<!-- Main modules, CLIs, services, how a request/data flows through the layers. -->
