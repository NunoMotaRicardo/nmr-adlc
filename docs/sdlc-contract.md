# The `.sdlc/` contract

## Purpose

The lifecycle skills (pm, implement, pr-review, prototype, diagnose) contain no repo-specific values. Everything that differs per repo — GitHub identifiers, branch and worktree conventions, quality-gate commands, code layout — is read from a `.sdlc/` folder at the root of the repo the skill runs against (`git rev-parse --show-toplevel`; in a worktree, that is the worktree). The `setup` skill creates it from `templates/sdlc/`; this page is the reference.

Skills never guess or fall back to another repo's values. Where a file is missing the skill either stops and says the repo isn't configured, or skips the optional feature, as listed below.

## Files

| File | Required? | Read by | If missing |
|---|---|---|---|
| `sdlc-config.yml` | Yes | pm, implement (and its coder/reviewer/batch/pr-feedback briefs), pr-review, prototype | implement, reviewer and prototype stop ("repo isn't configured"); pm cannot resolve identifiers |
| `repo-structure.md` | Yes for code repos | every skill (the Folders table); implement (coder, feasibility gate), pm (`issues.md`), diagnose, pr-review, reviewer (the layout) | pm stops and tells the user; others lack the layout and lose accuracy |
| `tooling.md` | Recommended | implement (coder, reviewer), prototype, diagnose | no dependency-install step is known; fresh worktrees may fail to run gates |
| `quality-gates.md` | Recommended | pr-review, reviewer | pr-review assumes CI gates lint/types/tests; SAST handling unknown |
| `repos.yml` | Optional | pm | single-repo mode: the current repo is the sole issue target |
| `.github/workflows/wiki-sync.yml` (template `wiki-sync.yml`) | Optional | GitHub Actions | the `user_docs` folder is not mirrored to the GitHub wiki |

## Documentation folders

The skills write to three folders whose paths are set per repo in the **Folders** table of `repo-structure.md`. Skills refer to them as `<specs>`, `<pm_docs>` and `<user_docs>` and never hardcode a path.

| Key | Default | Holds | Written by | Read by |
|---|---|---|---|---|
| `specs` | `specs` | Technical specs: normative contracts, schemas, invariants (`contracts/`, `layering-rules.md`) | implement (coder), in the same PR as the code | every skill that checks behaviour against contracts |
| `pm_docs` | `docs` | Planning and design docs: `plans/`, `product/` (articles, `fdr/`), `architecture/` (`adr/`, `code/`), `sources/`, navigation in `Home.md` and `_Sidebar.md` | pm only (`plans/` exclusively) | brainstorm, implement, prototype, pr-review |
| `user_docs` | `wiki` | User documentation: install, configure, use | implement (coder), in the same PR as the user-facing change; pm checks it after merge | people; optionally mirrored to the GitHub wiki by `wiki-sync.yml` |

Any of them may be `none`. `pm_docs` and `user_docs` may be the same folder; syncing then publishes the planning docs too.

**Fallback.** If `repo-structure.md` is missing or has no Folders table (a repo configured before these keys existed), skills use the earlier layout: `specs` = `specs`, `pm_docs` = `wiki`, `user_docs` = `none`, and suggest running `setup` to choose the folders.

There is no bootstrap script: no skill references one. In multi-repo mode, clone each `repos.yml` entry as a sibling checkout yourself.

## `sdlc-config.yml` key reference

| Key | Req. | Used by | Meaning |
|---|---|---|---|
| `default_branch` | Yes | pm, implement, pr-review, prototype, reviewer | Branch worktrees start from and PRs target |
| `github.owner` | Yes | pm, implement | GitHub user/org; also `--owner` for `gh project` |
| `github.repository` | Yes | pm, implement | Repository name |
| `github.slug` | Yes | pm, implement, pr-review | `<owner>/<repo>`; used in every `gh api` path, `gh issue create --repo`, issue URLs |
| `github.project.number` | Optional | pm, implement | Board number for `gh project item-add` |
| `github.project.id` | Optional | pm, implement | Project node id (`PVT_...`) for `item-edit` |
| `github.project.status_field_id` | Optional | pm, implement | Id of the Status single-select field (`PVTSSF_...`) |
| `github.project.statuses.{backlog,ready,in_progress,in_review,done}` | Optional | pm, implement | Option ids of that field |
| `implement.worktree_path` | Yes for implement | implement, batch, pm, pr-review (mention) | Per-issue worktree path template; `{issue}`, `{slug}` |
| `implement.branch` | Yes for implement | implement, batch | Per-issue branch template; `{issue}`, `{slug}` |
| `implement.max_changed_loc` | Yes for implement | implement (feasibility gate), pm (issue slicing) | Max changed lines (tests included) for one issue |
| `prototype.worktree_path` | Yes for prototype | prototype | Worktree path template; `{slug}` |
| `prototype.branch` | Yes for prototype | prototype | Branch template; `{slug}` |
| `review.bots` | Optional | implement (`pr-feedback.md`) | Automated reviewer logins whose comments are triaged |
| `quality_gates.test` / `lint` / `format_check` / `type_check` | Yes for implement | coder, reviewer | Commands run verbatim; omit a key if the repo has no such gate |
| `wiki_sync.{enabled,source_path,workflow_path}` | Optional | none (documentation for people and the workflow) | Whether the `user_docs` folder is mirrored to the GitHub wiki; `source_path` equals `user_docs` |

If `github.project` is absent, all board moves are skipped silently and only issues are used.

## Single-repo vs multi-repo

**Single-repo (default).** No `repos.yml`. The repo is both the docs home (its `pm_docs` folder) and the only issue target; pm reads its own `sdlc-config.yml`.

**Multi-repo.** An umbrella (docs-home) repo carries `.sdlc/repos.yml`. pm resolves each target from the entry's `path`, as a sibling checkout of the umbrella, and reads **that target repo's own** `.sdlc/sdlc-config.yml`. Only `role: application` entries are routed to; `role: tooling` entries are never issue targets. A missing checkout is a hard error; pm never falls back to the umbrella's config. An issue is always filed in the repo whose PR closes it. Every target repo needs its own `.sdlc/` (at least `sdlc-config.yml`, `repo-structure.md`).

## GitHub Project ids

A board is optional. To fill `github.project`:

```bash
gh project list --owner <owner>                                   # find the number
gh project list --owner <owner> --format json                     # node ids (PVT_...)
gh project field-list <number> --owner <owner> --format json      # Status field id (PVTSSF_...) and option ids
```

In the `field-list` output, take the field named `Status`: its `id` is `status_field_id`, and each entry of its `options` maps to a `statuses.*` key by name. Recreating a project changes every id; re-run the command.

Token scopes: `gh project` needs `read:project` and `write:project`:

```bash
gh auth refresh -s read:project -s write:project
```

Skills never read tokens from files.
