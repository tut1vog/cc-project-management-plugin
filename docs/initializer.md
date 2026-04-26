# initializer

A senior Claude Code specialist agent that helps you think through a new project carefully, then writes the full project scaffolding so director can plan and execute the setup.

## What it does

- Runs structured requirements discovery across seven phases before producing anything
- Challenges vague scope, unrealistic choices, and cargo-culted tech decisions
- Validates tech choices via WebSearch (library maintenance, runtime versions, community best practices)
- Introduces Claude Code features one at a time as they become relevant
- Consults the bundled `skillex` MCP server during Phase 6 to surface pre-built skills from an external catalog (default: `anthropics/skills`) as candidate features
- Runs `git init` (if needed) and halts on a dirty tree so every scaffold write is cleanly diffable
- Adds `*.local.*` to `.gitignore` so `CLAUDE.local.md` (and any other `*.local.*` Claude Code conventions) stays out of git
- Writes the project scaffolding: `CLAUDE.md` (long-term project facts, auto-loaded), `CLAUDE.local.md` (current goal — auto-loaded; gitignored), `.claude/rules/*.md`, and `.claude/settings.local.json` (director permissions; gitignored per-developer, or `.claude/settings.json` if you want them committed)

## Usage

Invoke the initializer by name in Claude Code. It always runs Discovery first — it will not start writing scaffolding until requirements are confirmed.

### Start a new project

```
Use initializer to set up this project
```

The initializer will walk you through seven phases of discovery, then — after you approve the Requirements Summary — write the scaffolding (`CLAUDE.md`, `CLAUDE.local.md`, `.claude/rules/*.md`, `.claude/settings.local.json`).

### After the initializer finishes

Once scaffolding is written, invoke director to plan and execute the setup. `CLAUDE.local.md` is auto-loaded by Claude Code, so director picks up the current goal automatically:

```
Use director to plan and execute the project
```

## Discovery phases

The initializer asks one phase at a time, summarizing and confirming before moving on.

| Phase | What it covers |
|-------|----------------|
| 1. Problem space | One-sentence description, who the users are, what problem it solves |
| 2. Scope and constraints | Hard constraints (language, platform, team, compliance, budget), MVP vs. deferred, known unknowns |
| 3. Technical direction | Language/runtime choice and rationale, external dependencies, deployment, testing strategy — validated via WebSearch |
| 4. Collaboration | Team size and roles, branching/review workflow, long-term ownership |
| 5. Standards | Linting/formatting, naming/commit conventions, license |
| 6. Claude Code setup | Which Claude Code features fit the project: CLAUDE.md, rules, commands, agents, settings, MCP, hooks — plus pre-built skills surfaced by consulting the bundled `skillex` MCP server against the Discovery answers |
| 7. Director permissions | Establishes what the director and subagents may do autonomously (bash commands, file ops, git, network, packages, destructive operations) |

### Requirements Summary

After all seven phases, the initializer produces a summary covering: project name/users/problem, constraints, MVP/deferred scope, stack, deployment, testing, team workflow, license, known unknowns, the concrete Claude Code setup it will write in Scaffold (`CLAUDE.md` sections, `CLAUDE.local.md` sections, each `.claude/rules/*.md` file and its trigger sentence, `.claude/settings.local.json` permission derivation), and the filled-in Phase 7 Director Permissions table (Bash allow/deny, file creation, protected paths, git commits, network access, package management, always-confirm operations).

Nothing is written until you approve this summary.

## Scaffold

Once approved, the initializer:

1. Ensures a git repo exists (`git init` if needed) and halts on a dirty tree to ask about commit/stash/abort. Adds `*.local.*` to `.gitignore` so the `CLAUDE.local.md` written next stays untracked.
2. Writes `CLAUDE.md` at the project root (stack, directory layout, canonical commands, constraints, and a Rules index listing each `.claude/rules/*.md` file with a one-line trigger — rules are loaded on demand, not auto-imported) and `CLAUDE.local.md` (project overview, scope, director permissions, known unknowns) — both auto-loaded by Claude Code at session start.
3. Writes each `.claude/rules/*.md` file from the approved list, each with a focused "when to read this" trigger and project-specific content.
4. Writes `.claude/settings.local.json` translating the Phase 7 permissions into `permissions.allow` / `permissions.deny` / `defaultMode`.

There is no separate handoff write — `CLAUDE.local.md` is the planning input director consults, and Claude Code auto-loads it whenever director runs.

## When to use this vs. other agents

| Situation | Agent |
|-----------|-------|
| **New project, no code yet** | initializer -> director |
| **Existing project, needs Claude Code setup** | advisor |
| **Multi-step implementation work** | director |

## Tips

- **Don't skip Discovery.** The initializer will challenge vague requirements — this is by design. Better to resolve ambiguity before planning begins.
- **Be specific about constraints.** The more the initializer knows about your team, platform, and compliance requirements, the more tailored the handoff.
- **You can say no.** The initializer proposes Claude Code features but doesn't force them. Decline anything that doesn't fit.
