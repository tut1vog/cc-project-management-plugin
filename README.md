# Claude Code Project Management Plugin

A Claude Code plugin that manages and automates Claude Code projects — from initial setup through ongoing development. Install it once and get the `director` orchestration subagent plus the `plan-management` and `project-scaffolding-context` skills, for structured planning, execution, and verification across every project you work in.

## Prerequisites

* **Claude Code with plugin support.** You need a version of Claude Code that recognizes the `/plugin` slash commands (`/plugin marketplace add`, `/plugin install`).

## Installation

Run these two slash commands inside a Claude Code session:

```
/plugin marketplace add tut1vog/cc-project-management-plugin
/plugin install cc-project-management-plugin@tut1vog-plugins
```

The first command registers this repo as a single-plugin marketplace named `tut1vog-plugins`. The second installs the `cc-project-management-plugin` plugin from that marketplace. Once installed, the agents are globally available — Claude Code auto-selects them by their `description` fields, so you can invoke them in any project without copying any files.

To pick up a new version later, re-run `/plugin install cc-project-management-plugin@tut1vog-plugins` (or use whatever update command your Claude Code version provides).

## Getting started

To set up Claude Code for a project, ask director (or any agent) to set up the project — it will load the `project-scaffolding-context` skill and run Discovery: a short interview to capture the goal, stack, conventions, rules, and permissions, then write `CLAUDE.md`, `CLAUDE.local.md`, rule files, and `.claude/settings.local.json`.

Once setup is done, director plans and executes the work. You can launch director at any time with a new goal — a feature, a refactor, a bug fix — and it will plan, dispatch, verify, and commit until the goal is met.

```
claude --agent cc-project-management-plugin:director
```

### Project documentation

The `project-scaffolding-context` skill asks where project documentation lives during Discovery (e.g. `docs/`) and writes a path-scoped `.claude/rules/documentation.md` capturing the conventions to follow. Director then writes entries to that folder after passed tasks that introduce architecturally significant content — a new third-party dependency, external API integration, persisted data shape, or captured design decision — without prompting on every write; the loaded rule constrains the path and style.

Opt out during Discovery to skip the rule and the maintenance step entirely.

## Agents

### director

Autonomous technical director for complex multi-step work. Acts as a powerful actor — reads files, runs commands, edits code, and delegates to subagents only when warranted. Operates a continuous sense-act loop: reads the situation, picks the next action, executes, repeats until the goal is met. Owns all git commits.

## Skills

The plugin bundles two skills:

- **`plan-management`** — canonical format and read/write instructions for the `PLAN.md` file director maintains at the repo root. Loaded by director on demand when the goal warrants persisting a plan.
- **`project-scaffolding-context`** — context for setting up Claude Code in a project: what information a scaffold requires, what files result, and their templates. User-invocable — load it before asking to scaffold a project or be grilled about project setup.

