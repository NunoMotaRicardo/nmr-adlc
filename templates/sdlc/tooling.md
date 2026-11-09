# Tooling — <repo>

<!-- Read by /implement (coder, reviewer), /prototype and /diagnose. A fresh git worktree has no
     installed dependencies, so the setup step below is run first in every worktree. -->

## Setup (run in every fresh worktree)

```bash
<install command, e.g. uv sync --extra dev | npm ci | go mod download | cargo fetch | mvn -q dependency:resolve>
```

## Running things

- Run the app / services: `<command>`
- Query the data store or call the service directly (used by /diagnose to reproduce symptoms): `<command>`
- Required tools and versions: <runtime, package manager, CLIs>

## GitHub

- Board / project: <number and name, or "none">. Ids live in `sdlc-config.yml`; re-derive with
  `gh project field-list <n> --owner <owner> --format json` if a call 404s.
- Auth: `gh auth status`; board moves need `gh auth refresh -s read:project -s write:project`.

## Secrets & environment

<!-- Names of required environment variables and where they come from. NEVER put values here. -->
