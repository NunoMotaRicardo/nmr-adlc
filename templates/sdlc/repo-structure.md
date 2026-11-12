# Repo structure — <repo>

<!-- Read by /implement (coder, feasibility gate), /pm (issue slicing), /diagnose, /pr-review and the
     reviewer. They never assume a layout: this file is where they learn it. Replace every <...>.
     If this file is missing, /pm and /implement stop and say the repo isn't configured. -->

## Folders

<!-- The three documentation folders the skills write to, as paths relative to the repo root, without
     a trailing slash. Every skill resolves `<specs>`, `<pm_docs>` and `<user_docs>` from this table,
     so change a folder here, never in the skills. Keep the keys exactly as written; `/setup` fills the
     values. Use `none` for a folder the repo doesn't have. -->

| Key | Folder | Holds | Written by |
|---|---|---|---|
| `specs` | `specs` | Technical specs: normative contracts, schemas and invariants (`contracts/`, `layering-rules.md`, ...) | /implement, in the same PR as the code |
| `pm_docs` | `docs` | Planning and design docs: `plans/`, `product/` (articles, FDRs), `architecture/` (ADRs, code docs), `sources/` | /pm |
| `user_docs` | `wiki` | User documentation: how to install, configure and use what the repo ships | /implement, in the same PR as the user-facing change |

User docs mirrored to the GitHub wiki: <yes, by `.github/workflows/wiki-sync.yml` | no>.

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
- Specs, planning docs and user docs: see **Folders** above.
- Data at rest (if any), and how to query it: <path or mechanism>

## Entry points

<!-- Main modules, CLIs, services, how a request/data flows through the layers. -->
