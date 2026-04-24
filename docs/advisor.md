# advisor

A senior software architect agent that audits an existing project, recommends Claude Code setup improvements, then writes the refreshed scaffolding and a handoff document for director to plan and execute further changes.

## What it does

- Silently reads the entire project before asking a single question (Audit)
- Surfaces gaps in Claude Code setup, conventions, and dependency health
- Validates tech choices and dependency maintenance status via WebSearch
- Walks through targeted discovery focused only on what it couldn't infer from code
- Runs `git init` (if needed) and halts on a dirty tree before touching any files
- Overwrites `CLAUDE.md`, `.claude/rules/*.md`, and `.claude/settings.json` with refreshed versions (git is the audit trail — no backup files, no diff prompts)
- Writes a narrowed `project-brief.md` as the handoff for director

## Installation

Copy `.claude/agents/advisor.md` from this repo into your project's `.claude/agents/` directory. No other setup is required.

```
your-project/
  .claude/
    agents/
      advisor.md   <- copy here
```

## Usage

Invoke the advisor by name in Claude Code. It always runs a silent audit first — reading your project files before presenting findings.

### Audit an existing project

```
Use advisor to review this project's Claude Code setup
```

The advisor will:
1. Silently read your project (manifests, README, existing Claude Code files, configs, CI, source structure)
2. Present an Audit Summary
3. Walk through targeted discovery (only asking about gaps)
4. Produce a Requirements Summary for your approval
5. Ensure git is initialized and the working tree is clean (halt if not)
6. Write / overwrite `CLAUDE.md`, `.claude/rules/*.md`, `.claude/settings.json`, and `project-brief.md`

### After the advisor finishes

Once `project-brief.md` is written, invoke director to plan and execute the improvements:

```
Use director — read project-brief.md for the project requirements
```

## Audit Summary

The advisor reads these areas silently before asking anything:

| Area | What it checks |
|------|----------------|
| Project identity | README, manifest files (package.json, pyproject.toml, Cargo.toml, etc.) |
| Claude Code presence | CLAUDE.md, .claude/rules/, .claude/commands/, .claude/agents/, .claude/settings.json |
| Documentation state | docs/, LICENSE, and any other documentation files |
| Coding conventions | Linting/formatting configs (.eslintrc, .prettierrc, pyproject.toml, rustfmt.toml, etc.) |
| Git workflow | .github/workflows/, PR templates, CODEOWNERS, Makefile CI targets |
| Source structure | Top-level directories, entry points |
| Dependency hygiene | Outdated or unmaintained dependencies (verified via WebSearch) |

## Discovery phases

Same seven phases as the initializer, but the advisor leads each phase with what it already observed and only asks about gaps:

| Phase | What it covers |
|-------|----------------|
| 1. Problem space | Confirms audit-derived understanding — "I see this is a [language] project that [purpose], is that accurate?" |
| 2. Scope and constraints | What is stable vs. planned, hard constraints not visible in code |
| 3. Technical direction | Validates current stack, flags outdated dependencies, confirms deployment and testing |
| 4. Collaboration | Team size, workflow, ownership |
| 5. Standards | Reviews existing conventions, linting, commit style, license |
| 6. Claude Code setup | Proposes features that fit, based on audit findings and discovery answers |
| 7. Director permissions | Establishes what the director and subagents may do autonomously (bash commands, file ops, git, network, packages, destructive operations) |

## When to use this vs. other agents

| Situation | Agent |
|-----------|-------|
| **New project, no code yet** | initializer -> director |
| **Existing project, needs Claude Code setup** | advisor -> director |
| **Multi-step implementation work** | director |

## Tips

- **Let the audit run.** The advisor reads everything silently first — this means fewer questions and more targeted advice.
- **Be specific about what's working.** The advisor won't touch things that are already fine.
- **You can say no.** The advisor proposes Claude Code features but doesn't force them. Decline anything that doesn't fit.
