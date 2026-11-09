# adlc — developer lifecycle plugin

Tool-neutral skills that take a change from idea to merged PR: `brainstorm`, `research`, `pm` (plans, docs, issues), `implement`, `pr-review`, `diagnose`, `prototype` and `setup`.

Works in any repo. Repo-specific values (GitHub owner/repo, default branch, quality-gate commands, optional project board) live in the target repo's `.sdlc/` folder. Run the `setup` skill once per repo to create it; see `docs/sdlc-contract.md` for the full reference.

Install through the [nmr-plugins](https://github.com/NunoMotaRicardo/nmr-plugins) marketplace (Claude Code, Codex, Copilot CLI, Cursor) or directly from this repo (Gemini CLI, Antigravity).

## Install (Claude Code)

```bash
claude plugin marketplace add NunoMotaRicardo/nmr-plugins
claude plugin install adlc@nmr-plugins
```
