---
name: setup
description: >
  Configure a repository for the developer-lifecycle skills by creating its .sdlc/ folder
  (sdlc-config.yml, repo-structure.md, quality-gates.md, tooling.md, optional repos.yml) and,
  optionally, a wiki/ folder with the wiki-sync workflow. Use on first use in a repo, when
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
- Existing `.sdlc/`, `wiki/`, `specs/`, `CLAUDE.md`, `.github/workflows/` — note what is already there.

Report what you found and confirm `default_branch` and `github.{owner,repository,slug}` one at a time.

### 2. Detect the ecosystem and propose quality gates
Look at manifests at the root (and one level down for monorepos): `package.json`, `pyproject.toml`, `uv.lock`, `requirements*.txt`, `go.mod`, `Cargo.toml`, `pom.xml`, `build.gradle*`, `*.csproj`/`*.sln`, `Gemfile`, `composer.json`. Read the relevant config (npm `scripts`, `[tool.*]` in pyproject, CI workflow under `.github/workflows/`) and propose `quality_gates.test`, `lint`, `format_check`, `type_check` — preferring the exact commands CI already runs. Use the examples in the template for the ecosystem when nothing is configured. Leave a gate out rather than inventing one the repo doesn't have. Confirm each command, then the install step for `tooling.md`.

### 3. Remaining values
Confirm, one at a time, with these defaults: `implement.worktree_path` `.worktrees/issue-{issue}`, `implement.branch` `issue-{issue}-{slug}`, `implement.max_changed_loc` `500`, `prototype.worktree_path` `.worktrees/prototype-{slug}`, `prototype.branch` `prototype-{slug}`, `review.bots` (ask which automated reviewers comment on PRs; empty is fine). Offer to add `.worktrees/` to `.gitignore`.

### 4. Project board (optional)
Ask whether the user wants a GitHub Project board. If not, omit `github.project`. If yes: ask for the board number (or run `gh project list --owner <owner>`), then `gh project list --owner <owner> --format json` for the node id and `gh project field-list <number> --owner <owner> --format json` for the `Status` field id and its option ids; map option names to `backlog`, `ready`, `in_progress`, `in_review`, `done` and confirm the mapping. On `missing required scopes`, tell the user to run `gh auth refresh -s read:project -s write:project` themselves and retry after they confirm.

### 5. Multi-repo (optional)
Ask whether this repo is an umbrella whose issues are filed in sibling repos. If so, build `repos.yml` from the template: one entry per repo, `role: application` or `tooling`, `path` as sibling directory. Remind the user that each target repo needs its own `.sdlc/` (offer to run this skill there) and that missing sibling checkouts must be cloned by hand. If not, do not create `repos.yml`.

### 6. Structure docs
Fill `repo-structure.md`, `quality-gates.md` and `tooling.md` from the templates using what you detected: directory layout, test locations, `specs/` presence, CI workflow, setup command. Mark anything you could not determine as a `<TODO: ...>` for the user rather than guessing.

### 7. Wiki (optional)
Ask whether to create `wiki/` (with `Home.md` and `_Sidebar.md`, the navigation `/pm` maintains) and mirror it to the GitHub wiki. If yes: create the folder with minimal placeholder pages, copy `<this skill's directory>/../../templates/sdlc/wiki-sync.yml` to `.github/workflows/wiki-sync.yml` (set the branch to `default_branch`), and set `wiki_sync.enabled: true`. Tell the user the one-time step: enable Wikis in the repo settings and create a first wiki page by hand. If no, leave `wiki_sync.enabled: false`.

### 8. Write and report
Write `.sdlc/sdlc-config.yml`, `repo-structure.md`, `quality-gates.md`, `tooling.md`, plus `repos.yml`, `wiki/`, workflow and `.gitignore` edit only if chosen, honouring the no-overwrite rule. Finish by listing every file created, modified or skipped, the `<TODO>` items still open, and that nothing was committed — the user reviews and commits.
