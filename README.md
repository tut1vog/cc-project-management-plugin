# Claude Code Project Subagents

A Claude Code plugin that manages and automates Claude Code projects — from initial setup through ongoing development. Install it once and get three orchestration subagents (`cc-project-initializer`, `cc-project-advisor`, `cc-project-director`) plus the `skillex-mcp` server for structured planning, execution, and verification across every project you work in.

## Prerequisites

- **Claude Code with plugin support.** You need a version of Claude Code that recognizes the `/plugin` slash commands (`/plugin marketplace add`, `/plugin install`).
- **Node ≥ 20** on your machine. The bundled `skillex-mcp` server is launched via `npx`, which requires a modern Node runtime. Check with `node --version`.
- **First-launch delay.** The very first Claude Code session after installing this plugin takes roughly 30–120 seconds to start, because `skillex-mcp`'s `prepare` script compiles TypeScript at install time. Subsequent launches are served from the `npx` cache and start fast.

## Installation

Run these two slash commands inside a Claude Code session:

```
/plugin marketplace add github:tut1vog/cc-project-management-plugin
/plugin install cc-project-management-plugin@tut1vog-plugins
```

The first command registers this repo as a single-plugin marketplace named `tut1vog-plugins`. The second installs the `cc-project-management-plugin` plugin from that marketplace. Once installed, the three agents are globally available — Claude Code auto-selects them by their `description` fields, so you can invoke them in any project without copying any files.

To pick up a new version later, re-run `/plugin install cc-project-management-plugin@tut1vog-plugins` (or use whatever update command your Claude Code version provides).

## Configuring skills

The plugin ships a `.mcp.json` that wires up the `skillex-mcp` server with `anthropics/skills` as the baked-in default skill repository. You don't need to configure anything for the default behavior to work.

If you want to point skillex at a different set of repositories, set the `SKILLS_MCP_REPOS` environment variable before launching Claude Code.

Two important details about how this variable works:

- **It is comma-separated.** List multiple repos as `owner1/repo1,owner2/repo2` with no spaces around the commas.
- **Your value replaces the default — it does not extend it.** Skillex has no append semantics. If you want to keep `anthropics/skills` while adding your own, you must include `anthropics/skills` explicitly in your list.

Examples:

```bash
# one-off: set for a single session
SKILLS_MCP_REPOS="anthropics/skills,myorg/my-skills" claude

# persistent: export in a shell profile (e.g. ~/.bashrc or ~/.zshrc)
export SKILLS_MCP_REPOS="anthropics/skills,myorg/my-skills"
```

## The Agents

### cc-project-initializer

Guides you through setting up a **new project** from scratch. Runs a structured discovery conversation (problem space, constraints, tech stack, collaboration workflow, standards), recommends which Claude Code features to use, and establishes **director permissions** — what the director and its subagents may do autonomously versus what requires your confirmation (shell commands, file operations, git, network access, package management, destructive operations). Then writes the full project scaffolding: `CLAUDE.md` (durable shared context), `.claude/rules/*.md` (on-demand behavioral rules), `.claude/settings.json` (permissions), and `project-brief.md` (planning input). Runs `git init` first if the working directory isn't a git repo, so every scaffold write is cleanly diffable and revertible.

### cc-project-advisor

Same role as the initializer, but for **existing projects**. Silently audits the codebase first (README, manifests, linting configs, CI, Claude Code setup), then walks through targeted discovery that only asks about gaps the audit couldn't fill — including **director permissions** so execution runs smoothly without repeated permission prompts. Writes the same scaffolding as the initializer (`CLAUDE.md`, `.claude/rules/*.md`, `.claude/settings.json`, `project-brief.md`), overwriting existing files where needed. Halts on a dirty git tree and asks to commit, stash, or abort before touching anything — git is the audit trail, so there are no backup files or diff prompts.

### cc-project-director

The orchestrator. Reads `project-brief.md` (or any goal you give it), breaks it into phases and tasks, and runs an autonomous loop — dispatching work to subagents, verifying results, and advancing the plan until complete. Maintains `plan.md` as working memory and records task outcomes in git commit message bodies, so `git log` is the long-term journal. It never writes application code — it plans, dispatches, and verifies. Subagents never make git commits; the director owns all git operations: a passed task is one commit bundling code + docs + `plan.md`, a failed task or supersession is a `chore(ai):` commit of the `plan.md` revision, and pure bookkeeping events use `git commit --allow-empty` so every outcome lands in `git log`. On verification pass, the director also updates project documentation to reflect the verified change and bundles those edits into the same commit so docs never drift from reality. Behavioral rules (git conventions, testing, documentation) live in `.claude/rules/*.md`; the director reads them on demand based on the task at hand, using the Rules index in `CLAUDE.md`. Implementation code and tests are always separate tasks assigned to different agents, so the test author approaches the code as a consumer rather than the original author. The director auto-continues through its loop, only stopping when presenting a new plan for confirmation or when a subagent reports a task as infeasible.

## How They Work Together

The agents form a pipeline: **discover intent → plan work → execute and verify**.

### New project

```
You ──→ cc-project-initializer ──→ project-brief.md ──→ cc-project-director ──→ subagents do the work
         (discovery conversation)                        (plan → dispatch → verify loop)
```

1. Invoke **cc-project-initializer**. It asks about your project goals, constraints, stack, team workflow, and standards. It recommends Claude Code features (CLAUDE.md, rules, commands, agents, hooks, MCP servers) based on your answers. Finally, it walks through **director permissions** — which shell commands, file operations, git actions, and other operations the director and its subagents may perform autonomously, and which require your confirmation.
2. Once you approve the requirements summary (including permissions), it writes the scaffolding: `CLAUDE.md`, `.claude/rules/*.md`, `.claude/settings.json`, and `project-brief.md`. It runs `git init` first if needed so everything lands in git history.
3. Invoke **cc-project-director** and tell it to read `project-brief.md`. It decomposes the setup into phases and tasks, each assigned to the most appropriate subagent, then runs an autonomous dispatch → execute → verify loop until the plan is complete.

### Existing project

```
You ──→ cc-project-advisor ──→ project-brief.md ──→ cc-project-director ──→ subagents do the work
         (audit + discovery)                         (plan → dispatch → verify loop)
```

Same flow, but starts with **cc-project-advisor** instead. It reads your codebase first, then confirms what it found and asks about what it can't infer — including director permissions. After approval it writes the same scaffolding (`CLAUDE.md`, `.claude/rules/*.md`, `.claude/settings.json`, `project-brief.md`), overwriting any existing versions (it halts first if the git tree is dirty, so you can commit or stash before it touches anything). The director then takes over and runs autonomously.

### Ongoing development

Once the project is set up, you can invoke **cc-project-director** directly with any goal — a new feature, a refactor, a bug to investigate. It follows the same plan → dispatch → verify cycle, using `plan.md` and `git log` to maintain continuity across sessions.
