# cc-project-management-plugin

A Claude Code plugin bundling one orchestration subagent (`director`) and two skills (`plan-management`, `project-scaffolding-context`) — distributed as a single-plugin marketplace so users can install it globally with `/plugin install` instead of copying `.md` files into every project.

## Stack
- Language / runtime: **none at build time** — the repo is Markdown (agent definitions, rules, docs) and JSON (plugin manifests).
- No compile step, no test suite, no dependency lockfiles.

## Canonical Commands
- Local plugin test:       `claude --plugin-dir ./`
- Reload without restart:  `/reload-plugins` (inside a Claude Code session)
- Validate JSON manifest:  `python3 -m json.tool .claude-plugin/plugin.json`
- Validate marketplace:    `python3 -m json.tool .claude-plugin/marketplace.json`
- Release:                 `/release [major|minor|patch]`

There is no `build`, `test`, `lint`, or `run` target — Markdown + JSON only.

## Skills (bundled)
- `skills/plan-management/SKILL.md` — canonical format spec and read/write instructions for the `PLAN.md` file director maintains at the repo root. Read it before editing director's plan management behavior or any agent that needs to inspect plan state.
- `skills/project-scaffolding-context/SKILL.md` — context for setting up Claude Code in a project. Covers what information a scaffold requires, what files result, and their templates. User-invocable. Read it before editing the scaffold checklist, the CLAUDE.md template, or the reference template catalog.

## Permissions
Director's permissions for this repo live in the maintainer's local `.claude/settings.local.json` (gitignored).
