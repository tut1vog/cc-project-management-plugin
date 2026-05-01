# Claude Code Project Management Plugin

A Claude Code plugin that manages and automates Claude Code projects — from initial setup through ongoing development. Install it once and get three orchestration subagents (`scaffolder`, `director`, `investigator`) plus the `plan-management`, `project-scaffolding`, `skill-catalog`, and `prior-art-research` skills, for structured planning, execution, research, and verification across every project you work in.

## Prerequisites

* **Claude Code with plugin support.** You need a version of Claude Code that recognizes the `/plugin` slash commands (`/plugin marketplace add`, `/plugin install`).
* **Python 3** on your `PATH`. The bundled `bin/` helpers (`plan-management`, `skill-catalog`) use only the Python standard library.
* **`gh` CLI authenticated** *(optional — required only for the `skill-catalog` catalog search)*. Install from https://cli.github.com/ and run `gh auth login`, or set `GITHUB_TOKEN`. Without it, scaffolder skips catalog search and Discovery still completes.

## Installation

Run these two slash commands inside a Claude Code session:

```
/plugin marketplace add tut1vog/cc-project-management-plugin
/plugin install cc-project-management-plugin@tut1vog-plugins
```

The first command registers this repo as a single-plugin marketplace named `tut1vog-plugins`. The second installs the `cc-project-management-plugin` plugin from that marketplace. Once installed, the three agents are globally available — Claude Code auto-selects them by their `description` fields, so you can invoke them in any project without copying any files.

To pick up a new version later, re-run `/plugin install cc-project-management-plugin@tut1vog-plugins` (or use whatever update command your Claude Code version provides).

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

## The Agents

### scaffolder

Sets up Claude Code for a project or establishes the current goal — works for greenfield projects, for existing projects where Claude Code is missing or partial, and when you want to set or change the current goal on a project that already has Claude Code in place. Silently reads whatever code is already there (README, manifests, linting configs, CI, existing Claude Code setup), then walks through structured discovery (problem space, scope and goal, tech stack, collaboration, standards), recommends which Claude Code features to use, and establishes **director permissions** — what the director and its subagents may do autonomously versus what requires your confirmation (shell commands, file operations, git, network access, package management, destructive operations). Writes the scaffolding: `CLAUDE.md` (long-term project facts, auto-loaded), `CLAUDE.local.md` (current goal — auto-loaded; gitignored via `*.local.*`), `.claude/rules/*.md` (on-demand behavioral rules), and `.claude/settings.local.json` (director permissions; gitignored per-developer, or `.claude/settings.json` if you want them committed). Runs `git init` first if the directory isn't a git repo and adds `*.local.*` to `.gitignore`. Halts on a dirty git tree and asks to commit, stash, or abort before touching anything — git is the audit trail, so there are no backup files or diff prompts.

### director

The orchestrator. Reads `CLAUDE.local.md` (auto-loaded by Claude Code) for the current goal, breaks it into phases and tasks, and runs an autonomous loop — dispatching work to subagents, verifying results, and advancing the plan until complete. Stores the live plan in git history as empty `plan:` commits and records every task outcome in a `Task:` journal entry in the corresponding commit body, so `git log` is the single source of truth. It never writes application code — it plans, dispatches, and verifies. Subagents never make git commits; the director owns all git operations: a passed task is one commit bundling code + docs, a failed task or single-task supersession is a `chore(ai):` empty commit with the failure journal in the body, a plan creation or revision is an empty `plan:` commit, and pure bookkeeping events use `git commit --allow-empty` so every outcome lands in `git log`. On verification pass, the director also updates project documentation to reflect the verified change and bundles those edits into the same commit so docs never drift from reality. Behavioral rules (git conventions, testing, documentation) live in `.claude/rules/*.md`; the director reads them on demand based on the task at hand, using the Rules index in `CLAUDE.md`. Implementation code and tests are always separate tasks assigned to different agents, so the test author approaches the code as a consumer rather than the original author. The director auto-continues through its loop, only stopping when presenting a new plan for confirmation or when a subagent reports a task as infeasible.

### investigator

A research specialist. Investigates technical questions before action — bug root causes, feature design decisions, library or pattern choices, architectural alternatives — and reports a structured findings document without implementing the fix or feature. Director dispatches it for investigate-only tasks; the implementation lands as a separate task on a different agent, the same separation-of-concerns principle that keeps implementation and tests on different agents. The procedure lives in the `prior-art-research` skill, which the agent loads on every dispatch. Operates in three modes: **Understand** (extract the question's shape from the request, read implicated source, walk git history with `git log` / `git blame` / `git show`; for bugs: reproduce when feasible — temporary instrumentation like extra logs, `print` statements, or scratch scripts is allowed during this phase), **Research** (optional — look for prior art via `WebSearch`, `WebFetch`, and the `gh` CLI through `Bash` when the user has it authenticated; every external finding is cited by URL), and **Synthesize** (synthesize evidence into a single finding — for bugs: the most likely root cause classified as **logic** or **architectural**; for features/design: a recommended approach with rationale — and state a confidence level with what would raise it). The skill enforces a hard gate: if the question is trivial (a typo, an obvious off-by-one, a feature with clear in-codebase precedent), the agent exits without running and reports the answer directly. Closes out by reverting every instrumentation edit so `git status` shows a clean tree, then emits a structured Investigation Report that director pastes verbatim into the `Task:` journal entry. Never makes git commits, never implements (even for clear logic bugs or obvious feature recommendations), and never leaves the working tree dirty; for architectural causes and significant design choices, the report asks director to propose a separate phase to the user before any code change happens.

## The Skills

### plan-management

A bundled skill that owns the canonical format spec for `plan:` and `Task:` journal commit messages, plus the git commands to read and parse them. **Director uses it to write** new plan and task commits; **any other agent uses it to read** the current plan, derive task status (done / in-progress / pending), or look up a prior task's outcome — without needing to inline the format spec in its own prompt. Because the plan and task journal live entirely in `git log`, the skill needs only the `Bash` tool, so it works in any subagent that has shell access. Treat the skill as the single source of truth for the format: if it ever evolves, only one file changes.

### project-scaffolding

The canonical spec for the durable files **scaffolder** writes after Discovery — `CLAUDE.md`, `CLAUDE.local.md`, `.claude/rules/*.md`, `.claude/settings.local.json` — plus the Requirements Summary input contract and a catalog of hybrid rule templates filled from Discovery answers. Scaffolder reads it before writing the scaffolding; if you want to extend or audit what gets written, this is the file to look at.

### skill-catalog

Wraps `bin/skill-catalog`, the `gh`-backed helper that searches the configured trusted repositories for pre-built `SKILL.md` files. Scaffolder consults it during Phase 6 of Discovery to surface candidate skills for the project. Any agent with `Bash` access can run `skill-catalog search "<query>"` and `skill-catalog get <id>` directly.

### prior-art-research

The research-driven investigation procedure (Understand → Research → Synthesize) plus the unified findings-report format. The `investigator` agent loads it on every dispatch and treats it as authoritative. Any other agent can also load it directly when it needs to research a non-trivial bug, library or pattern choice, feature design, or architectural decision without spawning a fresh subagent. The skill includes a hard gate that exits without running for trivial questions (typos, obvious off-by-ones, features with clear in-codebase precedent), so the cost of broad availability is bounded.

## How They Work Together

The agents form a pipeline: **discover intent → plan work → execute and verify**.

### Setup or new goal

```
You ──→ scaffolder ──→ CLAUDE.local.md ──→ director ──→ subagents do the work
         (silent read + discovery)                 (plan → dispatch → verify loop)
```

1. Invoke **scaffolder**. It silently reads whatever code is already there (an empty directory just produces a short project summary), then walks through discovery — problem space, scope and goal, stack, collaboration, standards. It proposes Claude Code features based on what it found and what you answered, and walks through **director permissions** — which shell commands, file operations, git actions, and other operations the director and its subagents may perform autonomously, and which require your confirmation.
2. Once you approve the requirements summary (including permissions), scaffolder writes the scaffolding: `CLAUDE.md`, `CLAUDE.local.md`, `.claude/rules/*.md`, and `.claude/settings.local.json`. It runs `git init` first if needed and adds `*.local.*` to `.gitignore` so `CLAUDE.local.md` stays untracked. On a project with prior state it overwrites the same files; it halts first if the git tree is dirty, so you can commit or stash before it touches anything.
3. Invoke **director**. It picks up `CLAUDE.local.md` automatically (Claude Code auto-loads it), decomposes the goal into phases and tasks, each assigned to the most appropriate subagent, then runs an autonomous dispatch → execute → verify loop until the plan is complete.

### Ongoing development

Once the project is set up, you can invoke **director** directly with any goal — a new feature, a refactor, a bug to investigate. It follows the same plan → dispatch → verify cycle, using `plan:` commits and the `Task:` journal in `git log` to maintain continuity across sessions.
