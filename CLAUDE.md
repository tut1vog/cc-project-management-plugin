# cc-project-management-plugin

## Intention

Developers using Claude Code to build projects need a capable, opinionated agent in every session — one that acts directly, knows when to delegate to subagents, and knows when to commit. Claude Code's native agent has no explicit rules for either. This plugin solves that by distributing `director`, a ReAct agent whose system prompt encodes those rules, as a globally-installable Claude Code plugin. Instead of copy-pasting `.md` files into every project, users install once and get `director` everywhere.

The bundled skills extend `director`'s capabilities: managing a structured plan for complex tasks, initializing Claude Code projects following best practices, keeping documentation clean after agent edits, and doing web research to inform decisions.

## How the project works

This repo is pure Markdown and JSON — no build step, no test suite, no dependencies. Shipping a new behavior means editing the right `.md` file and bumping the version.

**Key files:**
- `agents/director.md` — the core artifact. The system prompt here is what users get in every session.
- `skills/plan-management/SKILL.md` — teaches director how to structure and track a multi-step plan in `PLAN.md`. Read before editing director's planning behavior.
- `skills/project-scaffolding/SKILL.md` — teaches director how to initialize a Claude Code project following best practices. Read before editing the scaffold checklist or CLAUDE.md templates.
- `skills/lint-instructions/SKILL.md` — cleans up documentation bad smells after an agent edits instruction files. Read before editing lint rules or bad-smell definitions.
- `skills/web-research/SKILL.md` — enables web search for decision-making and future reference. Read before editing research behavior.
- `.claude-plugin/plugin.json` — the plugin manifest. `version` here must match the release tag.
- `.claude/rules/` — authoring constraints that apply to every edit in this repo.

## Canonical Commands
- Local plugin test:       `claude --plugin-dir ./`
- Reload without restart:  `/reload-plugins` (inside a Claude Code session)
- Validate JSON manifest:  `python3 -m json.tool .claude-plugin/plugin.json`
- Validate marketplace:    `python3 -m json.tool .claude-plugin/marketplace.json`
- Release:                 `/release [major|minor|patch]`

## Constraints
- All agent and skill bodies must be project-agnostic — no hard-coded paths, no assumptions about the consumer's stack. See `.claude/rules/authoring.md`.
- Director owns all git commits in this repo. See `.claude/rules/git.md`.
- Permissions for this repo live in the maintainer's local `.claude/settings.local.json` (gitignored).
