# Claude Code Project Management Plugin

A Claude Code plugin that gives you `director` — a ReAct agent that replaces Claude Code's native agent with explicit rules for when to act directly, when to delegate to subagents, and when to commit. Install it once and get `director` plus four bundled skills (`plan-management`, `project-scaffolding`, `lint-instructions`, `web-research`) available across every project you work in, without copying any files.

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

Launch director with a goal:

```
claude --agent cc-project-management-plugin:director
```

To set up Claude Code for a new project, ask director to scaffold the project. It will run a short Discovery interview to capture the goal, stack, conventions, rules, and permissions, then write `CLAUDE.md`, `CLAUDE.local.md`, rule files, and `.claude/settings.local.json`.

For ongoing work, give director a goal — a feature, a refactor, a bug fix — and it will plan, execute, and commit until done.

## Agents

### director

Autonomous technical director for complex multi-step work. Acts as a powerful actor — reads files, runs commands, edits code, and delegates to subagents only when warranted. Operates a continuous sense-act loop: reads the situation, picks the next action, executes, repeats until the goal is met. Owns all git commits.

## Skills

The plugin bundles four skills:

- **`plan-management`** — format and write protocol for the `PLAN.md` file director maintains at the repo root. Director uses it to structure and track complex multi-step tasks.
- **`project-scaffolding`** — context for initializing a Claude Code project following best practices: what a scaffold requires, what files result, and their templates. User-invocable.
- **`lint-instructions`** — cleans up documentation bad smells (duplication, verbosity, ambiguity) after an agent edits instruction files. User-invocable.
- **`web-research`** — researches a topic online using web sources, GitHub, and official docs. Synthesizes findings inline or saves to disk. User-invocable.

