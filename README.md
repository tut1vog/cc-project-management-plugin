# Claude Code Project Management Plugin

A Claude Code plugin that manages and automates Claude Code projects — from initial setup through ongoing development. Install it once and get the `director` orchestration subagent plus the `plan-management`, `project-scaffolding`, `skill-catalog`, and `prior-art-research` skills, for structured planning, execution, research, and verification across every project you work in.

## Prerequisites

* **Claude Code with plugin support.** You need a version of Claude Code that recognizes the `/plugin` slash commands (`/plugin marketplace add`, `/plugin install`).
* **Python 3** on your `PATH`. The bundled `bin/` helpers (`plan-management`, `skill-catalog`) use only the Python standard library.
* **`gh` CLI authenticated** *(optional — required only for the `skill-catalog` catalog search)*. Install from https://cli.github.com/ and run `gh auth login`, or set `GITHUB_TOKEN`. Without it, the skill-catalog search step is skipped and Discovery still completes.

## Installation

Run these two slash commands inside a Claude Code session:

```
/plugin marketplace add tut1vog/cc-project-management-plugin
/plugin install cc-project-management-plugin@tut1vog-plugins
```

The first command registers this repo as a single-plugin marketplace named `tut1vog-plugins`. The second installs the `cc-project-management-plugin` plugin from that marketplace. Once installed, the agents are globally available — Claude Code auto-selects them by their `description` fields, so you can invoke them in any project without copying any files.

To pick up a new version later, re-run `/plugin install cc-project-management-plugin@tut1vog-plugins` (or use whatever update command your Claude Code version provides).

## Getting started

To set up Claude Code for a project, ask director (or any agent) to set up the project — it will load the `project-scaffolding` skill and run Discovery: a short interview to capture the goal, stack, conventions, rules, and permissions, then write `CLAUDE.md`, `CLAUDE.local.md`, rule files, and `.claude/settings.local.json`.

Once setup is done, director plans and executes the work. You can launch director at any time with a new goal — a feature, a refactor, a bug fix — and it will plan, dispatch, verify, and commit until the goal is met.

```
claude --agent cc-project-management-plugin:director
```

### Project documentation

The `project-scaffolding` skill asks where project documentation lives during Discovery (e.g. `docs/`) and writes a path-scoped `.claude/rules/documentation.md` capturing the conventions to follow. Director then writes entries to that folder after passed tasks that introduce architecturally significant content — a new third-party dependency, external API integration, persisted data shape, or captured design decision — without prompting on every write; the loaded rule constrains the path and style.

Opt out during Discovery to skip the rule and the maintenance step entirely.

## Agents

### director

Autonomous technical director for complex multi-step work. Acts as a powerful actor — reads files, runs commands, edits code, and delegates to subagents only when warranted. Operates a continuous sense-act loop: reads the situation, picks the next action, executes, repeats until the goal is met. Owns all git commits.

## Skills

The plugin bundles four skills:

- **`plan-management`** — canonical format and read/write instructions for the `PLAN.md` file director maintains at the repo root. Loaded by director on demand when the goal warrants persisting a plan.
- **`project-scaffolding`** — context for setting up Claude Code in a project: what information a scaffold requires, what files result, and their templates. User-invocable — load it before asking to scaffold a project or be grilled about project setup.
- **`skill-catalog`** — searches `SKILL.md` files in trusted GitHub repositories via the `gh` CLI. Auto-loaded by agents during project setup.
- **`prior-art-research`** — research procedure and report format for gathering evidence and synthesizing findings on technical questions. Available to any agent or director directly.

## Configuring skill-catalog

The `skill-catalog` skill searches GitHub for `SKILL.md` files in a list of trusted repositories. The default list is `["anthropics/skills"]` baked into `bin/skill-catalog`, so you don't need to configure anything for the default behavior to work.

To customize the list, create `~/.claude/skill-repos.json` — a JSON array of strings:

```json
[
  "anthropics/skills",
  "myorg/my-skills",
  "anotherorg/some-repo"
]
```

Each entry is either:

* `<owner>` — search every public `SKILL.md` under that GitHub user/org.
* `<owner>/<repo>` — search `SKILL.md` files only inside that repository.

The file is the full list (the built-in default is only used when the file is absent). To inspect what's covered, `cat ~/.claude/skill-repos.json`. The file lives in your home directory so plugin updates and reinstalls never touch it.
