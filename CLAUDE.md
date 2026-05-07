# cc-project-management-plugin

A Claude Code plugin bundling one orchestration subagent (`director`) and three skills (`plan-management`, `project-scaffolding-context`, `prior-art-research`) — distributed as a single-plugin marketplace so users can install it globally with `/plugin install` instead of copying `.md` files into every project.

## IMPORTANT
Only update version in plugin.json when we make a new release 

## Stack
- Language / runtime: **none at build time** — the repo is Markdown (agent definitions, rules, docs) and JSON (plugin manifests).
- No compile step, no test suite, no dependency lockfiles.

## Canonical Commands
- Local plugin test:       `claude --plugin-dir ./`
- Reload without restart:  `/reload-plugins` (inside a Claude Code session)
- Validate JSON manifest:  `python3 -m json.tool .claude-plugin/plugin.json`
- Validate marketplace:    `python3 -m json.tool .claude-plugin/marketplace.json`
- Release (tag + push):    `git tag v<semver> && git push origin main --tags`

There is no `build`, `test`, `lint`, or `run` target — Markdown + JSON only.

## Rules
- `.claude/rules/git.md` — applies to every commit and tag
- `.claude/rules/releasing.md` — applies when bumping the plugin version or cutting a release
- `.claude/rules/agent-authoring.md` — applies when creating or editing files under `agents/` or `skills/`
- `.claude/rules/no-legacy.md` — applies when editing any prose (agent prompts, skill prompts, rules, docs, READMEs)

## Skills (bundled)
- `skills/plan-management/SKILL.md` — canonical format spec and read/write instructions for the `PLAN.md` file director maintains at the repo root. Read it before editing director's plan management behavior or any agent that needs to inspect plan state.
- `skills/project-scaffolding-context/SKILL.md` — context for setting up Claude Code in a project. Covers what information a scaffold requires, what files result, and their templates. User-invocable. Read it before editing the scaffold checklist, the CLAUDE.md template, or the reference template catalog.
- `skills/prior-art-research/SKILL.md` — research-driven investigation procedure (Understand → Research → Synthesize) plus the unified findings-report format. Any agent — including director — can load it when researching a non-trivial bug, library/pattern choice, feature design, or architectural decision. Read it before editing the report format or the investigation procedure.

## Permissions
Director's permissions for this repo live in the maintainer's local `.claude/settings.local.json` (gitignored).
