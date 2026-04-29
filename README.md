# Claude Code Project Management Plugin

A Claude Code plugin that manages and automates Claude Code projects — from initial setup through ongoing development. Install it once and get two orchestration subagents (`advisor`, `director`), the `plan-management` skill that any agent can use to read or write the project plan, and the `skillex-mcp` server, for structured planning, execution, and verification across every project you work in.

## Prerequisites

* **Claude Code with plugin support.** You need a version of Claude Code that recognizes the `/plugin` slash commands (`/plugin marketplace add`, `/plugin install`).
* **Node ≥ 20** on your machine. The bundled `skillex-mcp` server is launched via `npx`, which requires a modern Node runtime. Check with `node --version`.
* **First-launch delay.** The very first Claude Code session after installing this plugin takes roughly 30–120 seconds to start, because `skillex-mcp`'s `prepare` script compiles TypeScript at install time. Subsequent launches are served from the `npx` cache and start fast.

## Installation

Run these two slash commands inside a Claude Code session:

```
/plugin marketplace add tut1vog/cc-project-management-plugin
/plugin install cc-project-management-plugin@tut1vog-plugins
```

The first command registers this repo as a single-plugin marketplace named `tut1vog-plugins`. The second installs the `cc-project-management-plugin` plugin from that marketplace. Once installed, the three agents are globally available — Claude Code auto-selects them by their `description` fields, so you can invoke them in any project without copying any files.

To pick up a new version later, re-run `/plugin install cc-project-management-plugin@tut1vog-plugins` (or use whatever update command your Claude Code version provides).

## Configuring skills

The plugin ships a `.mcp.json` that wires up the `skillex-mcp` server with `anthropics/skills` as the baked-in default skill repository. You don't need to configure anything for the default behavior to work.

If you want to point skillex at a different set of repositories, set the `SKILLS_MCP_REPOS` environment variable before launching Claude Code.

Two important details about how this variable works:

* **It is comma-separated.** List multiple repos as `owner1/repo1,owner2/repo2` with no spaces around the commas.
* **Your value replaces the default — it does not extend it.** Skillex has no append semantics. If you want to keep `anthropics/skills` while adding your own, you must include `anthropics/skills` explicitly in your list.

Examples:

```Shell
# one-off: set for a single session
SKILLS_MCP_REPOS="anthropics/skills,myorg/my-skills" claude

# persistent: export in a shell profile (e.g. ~/.bashrc or ~/.zshrc)
export SKILLS_MCP_REPOS="anthropics/skills,myorg/my-skills"
```

## The Agents

### advisor

Sets up Claude Code for a project or establishes the current goal — works for greenfield projects, for existing projects where Claude Code is missing or partial, and when you want to set or change the current goal on a project that already has Claude Code in place. Silently reads whatever code is already there (README, manifests, linting configs, CI, existing Claude Code setup), then walks through structured discovery (problem space, scope and goal, tech stack, collaboration, standards), recommends which Claude Code features to use, and establishes **director permissions** — what the director and its subagents may do autonomously versus what requires your confirmation (shell commands, file operations, git, network access, package management, destructive operations). Writes the scaffolding: `CLAUDE.md` (long-term project facts, auto-loaded), `CLAUDE.local.md` (current goal — auto-loaded; gitignored via `*.local.*`), `.claude/rules/*.md` (on-demand behavioral rules), and `.claude/settings.local.json` (director permissions; gitignored per-developer, or `.claude/settings.json` if you want them committed). Runs `git init` first if the directory isn't a git repo and adds `*.local.*` to `.gitignore`. Halts on a dirty git tree and asks to commit, stash, or abort before touching anything — git is the audit trail, so there are no backup files or diff prompts.

### director

The orchestrator. Reads `CLAUDE.local.md` (auto-loaded by Claude Code) for the current goal, breaks it into phases and tasks, and runs an autonomous loop — dispatching work to subagents, verifying results, and advancing the plan until complete. Stores the live plan in git history as empty `plan:` commits and records every task outcome in a `Task:` journal entry in the corresponding commit body, so `git log` is the single source of truth. It never writes application code — it plans, dispatches, and verifies. Subagents never make git commits; the director owns all git operations: a passed task is one commit bundling code + docs, a failed task or single-task supersession is a `chore(ai):` empty commit with the failure journal in the body, a plan creation or revision is an empty `plan:` commit, and pure bookkeeping events use `git commit --allow-empty` so every outcome lands in `git log`. On verification pass, the director also updates project documentation to reflect the verified change and bundles those edits into the same commit so docs never drift from reality. Behavioral rules (git conventions, testing, documentation) live in `.claude/rules/*.md`; the director reads them on demand based on the task at hand, using the Rules index in `CLAUDE.md`. Implementation code and tests are always separate tasks assigned to different agents, so the test author approaches the code as a consumer rather than the original author. The director auto-continues through its loop, only stopping when presenting a new plan for confirmation or when a subagent reports a task as infeasible.

## The Skill

### plan-management

A bundled skill that owns the canonical format spec for `plan:` and `Task:` journal commit messages, plus the git commands to read and parse them. **Director uses it to write** new plan and task commits; **any other agent uses it to read** the current plan, derive task status (done / in-progress / pending), or look up a prior task's outcome — without needing to inline the format spec in its own prompt. Because the plan and task journal live entirely in `git log`, the skill needs only the `Bash` tool, so it works in any subagent that has shell access. Treat the skill as the single source of truth for the format: if it ever evolves, only one file changes.

## How They Work Together

The agents form a pipeline: **discover intent → plan work → execute and verify**.

### Setup or new goal

```
You ──→ advisor ──→ CLAUDE.local.md ──→ director ──→ subagents do the work
         (silent read + discovery)                 (plan → dispatch → verify loop)
```

1. Invoke **advisor**. It silently reads whatever code is already there (an empty directory just produces a short project summary), then walks through discovery — problem space, scope and goal, stack, collaboration, standards. It proposes Claude Code features based on what it found and what you answered, and walks through **director permissions** — which shell commands, file operations, git actions, and other operations the director and its subagents may perform autonomously, and which require your confirmation.
2. Once you approve the requirements summary (including permissions), advisor writes the scaffolding: `CLAUDE.md`, `CLAUDE.local.md`, `.claude/rules/*.md`, and `.claude/settings.local.json`. It runs `git init` first if needed and adds `*.local.*` to `.gitignore` so `CLAUDE.local.md` stays untracked. On a project with prior state it overwrites the same files; it halts first if the git tree is dirty, so you can commit or stash before it touches anything.
3. Invoke **director**. It picks up `CLAUDE.local.md` automatically (Claude Code auto-loads it), decomposes the goal into phases and tasks, each assigned to the most appropriate subagent, then runs an autonomous dispatch → execute → verify loop until the plan is complete.

### Ongoing development

Once the project is set up, you can invoke **director** directly with any goal — a new feature, a refactor, a bug to investigate. It follows the same plan → dispatch → verify cycle, using `plan:` commits and the `Task:` journal in `git log` to maintain continuity across sessions.
