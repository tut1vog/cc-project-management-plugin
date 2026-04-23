# Project Brief

> Durable project facts (stack, commands, rules, conventions) live in `CLAUDE.md` and `.claude/rules/`. This brief is the planning input for cc-project-director: what we're building, what's already stable, what's planned, and how the director should operate on this repo.

## Project Overview

`cc-project-management-plugin` is a Claude Code plugin that packages three orchestration subagents (`cc-project-initializer`, `cc-project-advisor`, `cc-project-director`) plus the `skillex-mcp` server, distributed as a single-plugin GitHub-hosted marketplace so users can `/plugin install` instead of copy-pasting `.md` files into every project.

The audience is Claude Code users who want standardized project setup and autonomous execution available globally. Once installed, the three agents pipeline — discover intent → plan work → execute and verify — exactly as the `README.md` describes, now backed by skillex's fuzzy skill search.

## Current State

This repo just finished its plugin-ification scaffolding pass. Concretely:

- **Plugin structure in place**: `.claude-plugin/plugin.json` (v0.1.0), `.claude-plugin/marketplace.json` (marketplace name `tut1vog-plugins`), `.mcp.json` wiring skillex via `npx -y github:tut1vog/skillex-mcp` with `SKILLS_MCP_REPOS=anthropics/skills` as the shipped default.
- **Agents relocated** from `.claude/agents/` to `agents/` at the plugin root per the Claude Code plugin spec. Agent body content is unchanged from pre-scaffold.
- **LICENSE** (MIT, holder `tut1vog`, year 2026), **`.gitignore`** (ignores `.claude/settings.local.json`, `plan.md`, OS/editor noise).
- **Rules indexed in `CLAUDE.md`**: `git.md`, `releasing.md`, `agent-authoring.md`, `mcp-config.md`. Each has a trigger line; director reads on demand.
- **Git repo initialized**. First commit snapshotted the pre-scaffold state; scaffold work is uncommitted as of this brief.
- **skillex-mcp prerequisite landed**: `package.json` `prepare: "npm run build"` script committed and pushed to `tut1vog/skillex-mcp@main` by the user. `npx -y github:tut1vog/skillex-mcp` now works end-to-end.
- **`README.md` and `docs/`** untouched by scaffolding. README's "Installation" section still describes the old copy-paste workflow and must be rewritten before the plugin is published.
- **Three agent bodies** do not yet reference skillex tools (`search_skills`, `get_skill`, `list_repositories`). Wiring them to call the MCP server is explicitly deferred — the director will handle it post-scaffold.

## Constraints

- **Language / runtime**: Markdown + JSON only at build time. No compile step, no tests, no dependency lockfile in this repo.
- **Consumer runtime**: Node ≥ 20 is required on end-user machines for `npx` to launch skillex-mcp.
- **Distribution shape**: single-plugin repo (repo == marketplace), public on GitHub at `tut1vog/cc-project-management-plugin`. No multi-plugin refactor planned.
- **skillex-mcp is a live dependency**: its `main` branch is what consumers resolve against via `npx -y github:tut1vog/skillex-mcp`. A breaking change in skillex-mcp without a coordinated plugin update will break users. Treat it as a tightly coupled satellite.
- **No CI, no automated tests**: validation is manual (`python -m json.tool`, `claude --plugin-dir ./`, visual `/agents` and `/mcp` check).
- **Maintainer**: solo (`tut1vog`). Direct pushes to `main`.

## Scope

**Stable** (do not modify unless a task explicitly requires it):
- All three agent system prompts in `agents/*.md`. Content is considered correct; the plugin-ification was a location change only.
- `docs/*.md` — per-agent human-facing docs, unchanged through scaffolding.
- `LICENSE`, `.gitignore` as written.
- The plugin/marketplace structure decision (single-plugin repo, not multi-plugin).
- The MCP integration strategy (github source, `anthropics/skills` default, user overrides via env var).

**Planned** (tasks the director should schedule, roughly in this order):
1. **Rewrite `README.md`**. Replace the "Installation" section (currently describes copy-pasting `.md` files) with plugin install instructions: `/plugin marketplace add github:tut1vog/cc-project-management-plugin` → `/plugin install cc-project-management-plugin@tut1vog-plugins`. Add a "Prerequisites" section noting Node ≥ 20. Add a "Configuring skills" section explaining `SKILLS_MCP_REPOS` override semantics (replacement, not extension). Preserve the existing agent descriptions and "How They Work Together" sections.
2. **Smoke-test the plugin end-to-end**. Run `claude --plugin-dir ./`; confirm `/agents` lists all three and `/mcp` shows skillex running. Capture any failures as follow-up tasks. This should happen before the README rewrite is committed, so the README can describe a verified install path.
3. **Commit the scaffold**. Group the plugin structure (`.claude-plugin/`, `agents/` move, `.mcp.json`, `LICENSE`, `.gitignore`, `CLAUDE.md` refresh, `.claude/rules/*.md`, `project-brief.md`) into a logically-scoped set of Conventional Commits. Suggested shape: one `feat:` commit for the plugin structure, one `docs:` for CLAUDE.md + rules + brief, one `chore:` for LICENSE + .gitignore. Director decides exact grouping at dispatch time.
4. **Tag `v0.1.0` and push**. Follow `.claude/rules/releasing.md`. Both `git tag` and `git push` require user confirmation per local settings.
5. **Wire the agents to use skillex tools** (the reason skillex was bundled in the first place). Update each agent's system prompt to call `search_skills` / `get_skill` / `list_repositories` where appropriate — likely: the initializer and advisor consult skillex during Discovery to surface relevant skills as suggestions; the director consults it before dispatching an implementation task to check whether a matching skill already exists. Each agent edit is its own task, assigned to a separate subagent from whoever runs the subsequent verification. This is the largest remaining piece of work.
6. **Post-release housekeeping** (optional, director's call): `CHANGELOG.md`, a `CONTRIBUTING.md`, or a GitHub Actions workflow that validates JSON on PRs. Only schedule these if a specific need arises.

## Director Permissions

Managed locally by the maintainer in `.claude/settings.local.json` (gitignored). There is no committed `.claude/settings.json`. When running operations, the director relies on whatever `.claude/settings.local.json` provides; anything not explicitly allowed will prompt the user at runtime.

Baseline expectations the director should assume regardless of local settings:
- **Git commits**: director owns all commits; dispatched subagents never commit. Conventional Commits format per `.claude/rules/git.md`.
- **Git push / tag**: **always** require user confirmation on this public repo.
- **Destructive operations** (`rm -rf`, `git push --force`, tag deletion, deleting files under `agents/` or `.claude-plugin/`): **always** require user confirmation.
- **The sibling `skillex-mcp` repo at `/home/ubun/github.com/tut1vog/skillex-mcp`** is outside this project root. Any command that modifies it must be confirmed separately.

## Known Unknowns

- **Marketplace name ergonomics**: shipped as `tut1vog-plugins` so users install with `@tut1vog-plugins`. If that collides with an existing marketplace or feels awkward once published, it's a MINOR bump to change.
- **First-run `npx -y github:tut1vog/skillex-mcp` latency**: will vary by user network and npm cache state. Unmeasured. If it's painful enough to harm first-session UX, revisit by publishing `skillex-mcp` to npm and switching `.mcp.json` to the registry source (MINOR bump).
- **Whether `anthropics/skills` remains the right default index** as the Claude Code skills ecosystem evolves. Revisit periodically; default change is a MINOR bump.
- **How aggressively the three agents should call skillex** once wired (task 5 above). Over-calling adds latency and token cost; under-calling defeats the purpose of bundling it. Decide during that task's design, not now.
