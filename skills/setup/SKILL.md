---
name: setup
description: >
  Configure a repository for the developer-lifecycle skills by creating its .sdlc/ folder
  (sdlc-config.yml, repo-structure.md, quality-gates.md, tooling.md, optional repos.yml), choosing
  the specs, planning-docs and user-docs folders (and adapting an existing layout to them), and
  optionally mirroring the user docs to the GitHub wiki. Use on first use in a repo, when
  another skill reports the repo is "not configured" or `.sdlc/sdlc-config.yml` is missing,
  or when the user asks to set up, initialise or reconfigure the SDLC config.
---

# Setup

You create the per-repo `.sdlc/` contract the other skills read. You run in the **main thread**, interactively. The contract is defined in `<this skill's directory>/../../docs/sdlc-contract.md`; the templates are in `<this skill's directory>/../../templates/sdlc/` (`<this skill's directory>` is the folder containing this file; it is inside the plugin, so `../../` is the plugin root). Read the contract page and the templates before starting.

## Rules
- **Never read tokens from files** or ask for them. Use `gh` as already authenticated.
- **Never commit or push.** Leave everything as uncommitted working-tree changes.
- **Never overwrite an existing file without asking.** For each existing target, offer: skip, show a diff of what would change, or overwrite.
- Ask **one question at a time**, each with your recommended answer; accept "yes" as taking it.
- Do not invent values; every value comes from detection or the user. Keep the templates' comments in what you write.

## Steps

### 1. Detect the repo
- Root: `git rev-parse --show-toplevel`. Not a git repo → stop and say so.
- Default branch: `git symbolic-ref --short refs/remotes/origin/HEAD` (strip `origin/`), else `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`.
- Owner/repo: `gh repo view --json owner,name -q '.owner.login + "/" + .name'`, else parse `git remote get-url origin`. No remote → ask the user for the slug.
- Existing `.sdlc/`, `CLAUDE.md`/`AGENTS.md`, `.github/workflows/` (especially a wiki-sync workflow), and every candidate documentation folder: `specs/`, `docs/`, `wiki/`, `doc/`, `documentation/`, plus any folder holding `plans/`, `product/`, `architecture/` or `adr/`. Note what is already there; step 2 uses it.

Report what you found and confirm `default_branch` and `github.{owner,repository,slug}` one at a time.

### 2. Choose the documentation folders
The skills write to three folders, recorded in the **Folders** table of `repo-structure.md` (keys `specs`, `pm_docs`, `user_docs`; the contract page defines them):

| Key | Default | Holds |
|---|---|---|
| `specs` | `specs` | Technical specs: normative contracts, schemas, invariants. Written by /implement. |
| `pm_docs` | `docs` | Planning and design docs: `plans/`, `product/`, `architecture/`, `sources/`, plus `Home.md` and `_Sidebar.md`. Written by /pm. |
| `user_docs` | `wiki` | User documentation: install, configure, use. Optionally mirrored to the GitHub wiki. |

Ask for each folder in turn, one question at a time, with your recommendation. A folder may be `none` (a repo with no specs, or no user docs). `pm_docs` and `user_docs` may be the same folder only if the user wants the planning docs published with the user docs; say that this is what syncing would then do.

**Reconfiguring:** if `.sdlc/repo-structure.md` already has a **Folders** table, recommend its current values and only analyse folders that changed or don't exist yet.

**New repo, or none of the folders exist:** recommend the defaults.

**Existing repo:** analyse what is there before recommending, and classify each existing folder by its content rather than its name:
- A folder with `plans/`, `product/`, `architecture/`, `adr/`, FDRs or ADRs holds **planning docs**. The usual case is an older adlc layout where `wiki/` held all of them.
- A folder of how-to, install or reference pages for people using the software holds **user docs**. A site generator config (`mkdocs.yml`, `docusaurus.config.*`, `conf.py` for Sphinx, `book.toml`) marks a user-docs folder whose location is fixed by that tool.
- A folder of schemas, contracts or invariants holds **specs**.
- A folder that mixes these is **mixed**; list which files belong where.

Then present the findings as a short table (folder, what it holds, proposed role) and, for each folder that doesn't match its default, propose one of these actions and ask the user to pick:
- **Keep in place:** record the existing folder as the value. No files move. This is the recommendation when a tool or CI pins the path (site generator, published URLs, many inbound links).
- **Move to the default:** `git mv` the folder, or the listed files of a mixed folder, to the default location. Before proposing, list what the move breaks and include the fixes in the proposal: inbound links from `README.md`, `AGENTS.md`/`CLAUDE.md`, code comments and other docs (`git grep -n '<old>/'`); the `paths` filter and `rsync` source of an existing wiki-sync workflow; any CI, linter or site-generator config naming the old path. Relative links *inside* a folder that moves whole stay valid; links between folders that move apart don't — list them.
- **Split a mixed folder:** move the planning docs to `pm_docs` and leave the user docs where they are (or the reverse), with the same link check.

Typical case — an older adlc repo with `wiki/` holding `plans/`, `product/`, `architecture/` and mirrored to the GitHub wiki: recommend moving those planning folders, `Home.md` and `_Sidebar.md` to `docs/`, keeping `wiki/` for user docs (new placeholder `Home.md` and `_Sidebar.md`), and warning that the next sync removes the planning pages from the GitHub wiki. The alternative is keeping `pm_docs: wiki` and `user_docs: wiki`, which changes nothing.

If the chosen default name is already taken by unrelated content (e.g. `docs/` already holds user docs), say so and suggest another name for the other role instead of merging the two.

Collect the chosen actions; nothing moves until step 9.

### 3. Detect the ecosystem and propose quality gates
Look at manifests at the root (and one level down for monorepos): `package.json`, `pyproject.toml`, `uv.lock`, `requirements*.txt`, `go.mod`, `Cargo.toml`, `pom.xml`, `build.gradle*`, `*.csproj`/`*.sln`, `Gemfile`, `composer.json`. Read the relevant config (npm `scripts`, `[tool.*]` in pyproject, CI workflow under `.github/workflows/`) and propose `quality_gates.test`, `lint`, `format_check`, `type_check` — preferring the exact commands CI already runs. Use the examples in the template for the ecosystem when nothing is configured. Leave a gate out rather than inventing one the repo doesn't have. Confirm each command, then the install step for `tooling.md`.

### 4. Remaining values
Confirm, one at a time, with these defaults: `implement.worktree_path` `.worktrees/issue-{issue}`, `implement.branch` `issue-{issue}-{slug}`, `implement.max_changed_loc` `500`, `prototype.worktree_path` `.worktrees/prototype-{slug}`, `prototype.branch` `prototype-{slug}`, `review.bots` (ask which automated reviewers comment on PRs; empty is fine). Offer to add `.worktrees/` to `.gitignore`.

### 5. Project board (optional)
Ask whether the user wants a GitHub Project board. If not, omit `github.project`. If yes: ask for the board number (or run `gh project list --owner <owner>`), then `gh project list --owner <owner> --format json` for the node id and `gh project field-list <number> --owner <owner> --format json` for the `Status` field id and its option ids; map option names to `backlog`, `ready`, `in_progress`, `in_review`, `done` and confirm the mapping. On `missing required scopes`, tell the user to run `gh auth refresh -s read:project -s write:project` themselves and retry after they confirm.

### 6. Multi-repo (optional)
Ask whether this repo is an umbrella whose issues are filed in sibling repos. If so, build `repos.yml` from the template: one entry per repo, `role: application` or `tooling`, `path` as sibling directory. Remind the user that each target repo needs its own `.sdlc/` (offer to run this skill there) and that missing sibling checkouts must be cloned by hand. If not, do not create `repos.yml`.

### 7. Structure docs
Fill `repo-structure.md`, `quality-gates.md` and `tooling.md` from the templates using what you detected: the **Folders** table with the values chosen in step 2 (keep its keys exactly as in the template), directory layout, test locations, CI workflow, setup command. Mark anything you could not determine as a `<TODO: ...>` for the user rather than guessing.

### 8. Sync user docs to the GitHub wiki (optional)
Skip if `user_docs` is `none`. Otherwise ask whether to mirror the `user_docs` folder to the GitHub wiki. If an existing wiki-sync workflow is found, propose updating it rather than adding a second one.
- **Yes:** copy `<this skill's directory>/../../templates/sdlc/wiki-sync.yml` to `.github/workflows/wiki-sync.yml`, set the branch to `default_branch` and, if `user_docs` isn't `wiki`, replace the folder in the `paths` filter and the `rsync` source. Set `wiki_sync.enabled: true` and `wiki_sync.source_path` to the `user_docs` folder. Tell the user the one-time step: enable Wikis in the repo settings and create a first wiki page by hand.
- **No:** leave `wiki_sync.enabled: false`, still with `source_path` set to the `user_docs` folder.

Create any chosen folder that doesn't exist yet with minimal placeholder pages: `<pm_docs>/Home.md` and `<pm_docs>/_Sidebar.md` (the navigation /pm maintains), and `<user_docs>/Home.md` (plus `_Sidebar.md` when synced). Don't create `<specs>/`; /implement creates it with the first spec.

### 9. Write and report
Write `.sdlc/sdlc-config.yml`, `repo-structure.md`, `quality-gates.md`, `tooling.md`, plus `repos.yml`, new folders, workflow and `.gitignore` edit only if chosen, honouring the no-overwrite rule. Then apply the folder actions chosen in step 2: `git mv` the folders or files, and fix every link and path the move breaks, as listed in the proposal. Re-run `git grep -n '<old>/'` afterwards and report any reference left. Finish by listing every file created, moved, modified or skipped, the `<TODO>` items still open, and that nothing was committed — the user reviews and commits.
