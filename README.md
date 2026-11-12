# adlc — agentic planning and coding workflow

My day-to-day workflow for building software with AI agents, packaged as a plugin: from a half-formed idea, to a written plan, to GitHub issues, to a reviewed pull request.

I built it for myself, but it isn't tied to my projects. It works in any GitHub repo, and anyone is welcome to use it and [contribute](#contributing).

## The workflow

```
idea ─▶ brainstorm ─▶ research / prototype / diagnose ─▶ pm ─▶ implement ─▶ pr-review
                                                          │
                                         plans · docs · issues · board
```

| Skill | What it does |
|---|---|
| `brainstorm` | Interviews you one question at a time, with a recommended answer for each, until a diffuse idea becomes resolved work items. |
| `research` | Investigates a question against primary sources and returns cited findings. |
| `prototype` | Builds throwaway code to answer a design question, keeps the answer, deletes the code. |
| `diagnose` | Finds the cause of a defect or anomaly when the mechanism isn't known. |
| `pm` | Owns the backlog and its docs: keeps a plan up to date, writes it up in the planning docs, slices it into GitHub issues, and reconciles after merges. |
| `implement` | Takes one issue (or a batch) to a merge-ready PR: a feasibility gate, a coder, a reviewer (max 3 rounds), then drives every review comment to resolved. |
| `pr-review` | Reviews changes against the repo's standards and its specs and posts one real GitHub PR review with inline comments. |
| `setup` | Creates the repo-side configuration the other skills read. Run it once per repo. |

## Install

**Claude Code**

```bash
claude plugin marketplace add NunoMotaRicardo/nmr-plugins
claude plugin install adlc@nmr-plugins
```

**Codex, Copilot CLI, Cursor** — add the [nmr-plugins](https://github.com/NunoMotaRicardo/nmr-plugins) marketplace in your tool and install `adlc`.

**Gemini CLI, Antigravity** — install straight from this repo, e.g. `gemini extensions install https://github.com/NunoMotaRicardo/nmr-adlc`.

The skills are plain Agent Skills (`skills/<name>/SKILL.md`), so any tool that reads that format can use them. I develop against Claude Code; the other tools are supported on a best-effort basis.

## Getting started

1. Install the plugin.
2. In your repo, run the `setup` skill. It detects your default branch, GitHub owner and repo, and test/lint/type-check commands, asks where your docs live, asks you to confirm everything, and writes a `.sdlc/` folder. In an existing repo it looks at the folders you already have and proposes how to adapt them.
3. Start with `brainstorm`, or open a plan with `pm`.

Requirements: `git` and an authenticated [`gh`](https://cli.github.com/). The optional project board needs the `read:project` and `write:project` token scopes (`gh auth refresh -s read:project -s write:project`).

## How it is configured

Everything repo-specific lives in the target repo's `.sdlc/` folder, never in the skills: GitHub identifiers, branch and worktree conventions, quality-gate commands, an optional project board, and three documentation folders:

| Folder | Default | Holds |
|---|---|---|
| Technical specs | `specs/` | Contracts, schemas and invariants, updated with the code |
| Planning docs | `docs/` | Plans, product docs, decision records, architecture docs, owned by `pm` |
| User docs | `wiki/` | How to install, configure and use the software; optionally mirrored to the GitHub wiki |
 A single repo works out of the box; an optional `repos.yml` lets one planning repo route issues to several application repos. See [`docs/sdlc-contract.md`](docs/sdlc-contract.md) for the full reference and [`templates/sdlc/`](templates/sdlc/) for the templates.

## Contributing

Contributions are welcome: bug reports, ideas, and pull requests, especially for other AI tools and other ecosystems. Please open an issue first for anything large. Ground rules are in [`AGENTS.md`](AGENTS.md): keep skills tool-neutral, keep repo-specific values out of the skills, and keep the templates and the contract doc in sync with the config keys the skills read.

More documentation is on the [wiki](https://github.com/NunoMotaRicardo/nmr-adlc/wiki).

## License

[MIT](LICENSE)
