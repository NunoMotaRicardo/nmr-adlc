# nmr-adlc — developer lifecycle plugin

Tool-neutral skills for the full dev lifecycle (brainstorm → research → plan/PM → implement → PR review, plus diagnose, prototype and repo setup). Published via the `nmr-plugins` marketplace (https://github.com/NunoMotaRicardo/nmr-plugins) to Claude Code, Codex, Copilot CLI, Gemini CLI, Antigravity and Cursor.

`CLAUDE.md` is a symlink to this file. Edit `AGENTS.md` only.

## Principles
- Skills are Agent Skills folders (`skills/<name>/SKILL.md`); folder name == frontmatter `name`.
- Skills must work in any repo: no project names, owners, IDs or domain assumptions. Repo-specific values live in the target repo's `.sdlc/` (see `docs/sdlc-contract.md`).
- Tool-neutral wording: delegate to subagents where supported, otherwise run the brief inline.
- Never read tokens from files; on missing `gh` scopes, tell the user to run `gh auth refresh`.

## Layout
```
skills/<name>/          one skill each (+ supporting briefs next to SKILL.md)
templates/sdlc/         templates the `setup` skill copies into a target repo's .sdlc/
docs/sdlc-contract.md   reference for the .sdlc contract
plugin.json             portable manifest (Copilot, Codex, Antigravity)
.claude-plugin/         Claude Code manifest
.cursor-plugin/         Cursor manifest
gemini-extension.json   Gemini CLI manifest
wiki/                   documentation, mirrored to the GitHub wiki
```

## Conventions
- A config key used by a skill must appear in `templates/sdlc/sdlc-config.yml` and `docs/sdlc-contract.md`. Keep them in sync.
- The four manifests carry the same name/version/description; bump the version in all of them together and tag `vX.Y.Z`.
- Before release, grep for leaked project-specific names and IDs.

## Wiki
- `wiki/` is the source of truth; `Home.md` is the landing page. Do not edit the GitHub wiki directly: `wiki-sync` overwrites it on push to `main`.
- First-time setup: enable Wikis in repo settings and create one page by hand so the wiki repo exists.
