# scaffolder

A senior Claude Code specialist agent that sets up Claude Code for a project or establishes the current goal — reads any prior code, runs structured discovery, then writes the project scaffolding so director can plan and execute.

## What it does

- Silently reads the entire project before asking a single question (Phase 0) — an empty directory just produces a short project summary
- Surfaces gaps in Claude Code setup, conventions, and dependency health when prior state exists
- Validates tech choices and dependency maintenance status via WebSearch
- Walks through structured discovery — leading with what was read when prior state exists, asking open-ended when it does not
- Captures the current project goal in Phase 2 (Scope) — this is what director will work toward
- Consults the bundled `skill-catalog` skill during Phase 6 to surface pre-built skills (from Phase 0 findings and Discovery answers) as candidate features
- Runs `git init` (if needed) and halts on a dirty tree before touching any files
- Adds `*.local.*` to `.gitignore` so `CLAUDE.local.md` (and any other `*.local.*` Claude Code conventions) stays out of git
- Writes (or overwrites) `CLAUDE.md`, `CLAUDE.local.md`, `.claude/rules/*.md`, and `.claude/settings.local.json` (director permissions; gitignored per-developer, or `.claude/settings.json` if you want them committed) — git is the audit trail for the tracked files, no backup files, no diff prompts

## Usage

Invoke the scaffolder by name in Claude Code. It always reads the project silently first — reading whatever project files exist before presenting findings.

### Set up a new project

```
Use scaffolder to set up this project
```

### Review and improve an existing project

```
Use scaffolder to review this project's Claude Code setup
```

### Set or change the current goal

```
Use scaffolder to set a new goal for this project
```

In all three cases, scaffolder will:
1. Silently read your project (manifests, README, existing Claude Code files, configs, CI, source structure) — fast and short on an empty directory
2. Present a Project Summary
3. Walk through the seven discovery phases — confirming what it observed when prior state exists, asking open-ended when it does not
4. Produce a Requirements Summary for your approval
5. Ensure git is initialized and the working tree is clean (halt if not), and add `*.local.*` to `.gitignore`
6. Write or overwrite `CLAUDE.md`, `CLAUDE.local.md`, `.claude/rules/*.md`, and `.claude/settings.local.json`

### After the scaffolder finishes

Once scaffolding is written, invoke director to plan and execute the work. `CLAUDE.local.md` is auto-loaded by Claude Code, so director picks up the current goal automatically:

```
Use director to plan and execute the work
```

## Project Summary

The scaffolder reads these areas silently before asking anything:

| Area | What it checks |
|------|----------------|
| Project identity | README, manifest files (package.json, pyproject.toml, Cargo.toml, etc.) |
| Claude Code presence | CLAUDE.md, CLAUDE.local.md, .claude/rules/, .claude/commands/, .claude/agents/, .claude/settings.json, .claude/settings.local.json |
| Documentation state | docs/, LICENSE, and any other documentation files |
| Coding conventions | Linting/formatting configs (.eslintrc, .prettierrc, pyproject.toml, rustfmt.toml, etc.) |
| Git workflow | .github/workflows/, PR templates, CODEOWNERS, Makefile CI targets |
| Source structure | Top-level directories, entry points |
| Dependency hygiene | Outdated or unmaintained dependencies (verified via WebSearch) |

## Discovery phases

Seven phases, asked one at a time. When Phase 0 found prior state, each phase leads with what was observed; when it found nothing, each phase is asked open-ended.

| Phase | What it covers |
|-------|----------------|
| 1. Problem space | One-sentence description, users, problem solved — confirms Phase 0 understanding when prior state exists |
| 2. Scope and constraints | Captures the current goal: stable + planned for projects with prior state, MVP + deferred for greenfield; plus hard constraints not visible in code |
| 3. Technical direction | Validates the Phase 0-detected stack via WebSearch, or asks for the chosen language/runtime, dependencies, deployment, and testing strategy and validates the choices |
| 4. Collaboration | Team size, branching workflow, long-term ownership |
| 5. Standards | Linting/formatting, CI enforcement, naming and commit conventions, license |
| 6. Claude Code setup | Proposes features that fit, based on Phase 0 findings and discovery answers — plus pre-built skills surfaced by consulting the bundled `skill-catalog` skill against those findings |
| 7. Director permissions | Establishes what the director and subagents may do autonomously (bash commands, file ops, git, network, packages, destructive operations) |

## When to use this vs. other agents

| Situation | Agent |
|-----------|-------|
| **Project setup, review, or goal change** | scaffolder -> director |
| **Multi-step implementation work** | director |

## Tips

- **Let Phase 0 run.** The scaffolder reads everything silently first — this means fewer questions and more targeted advice on a project that has prior state.
- **Be specific about scope.** The Scope you confirm in Phase 2 is the current goal director will work toward, so it pays to be precise about what's in versus out.
- **You can say no.** The scaffolder proposes Claude Code features but doesn't force them. Decline anything that doesn't fit.
