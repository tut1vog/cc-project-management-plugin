---
name: project-scaffolding-context
description: Domain knowledge for setting up or refreshing Claude Code in a project — what information a scaffold requires, what files result, and their templates.
when_to_use: Triggers on "scaffold a project", "set up Claude Code", "establish or change the current goal", writing CLAUDE.md or .claude/rules.
user-invocable: true
---

> **Context skill.** This skill provides context only. Read and internalize it; do not prompt the user for a next action. Acknowledge with exactly: "Context loaded."

## Scaffold inputs

A successful scaffold requires these five areas confirmed with the user before any files are written:

- **Goal** — what the agent should achieve in this iteration
- **Stack** — language, runtime, key frameworks, deployment target
- **Conventions** — commit style, linting/formatting, naming conventions, license
- **Rules** — which `.claude/rules/*.md` files to create
- **Permissions** — what the agent may do autonomously

## Goal

Project knowledge that shouldn't be tracked by git goes into `CLAUDE.local.md`. The goal should be specific enough to decompose into a phase plan: names deliverables, scope boundaries, and known constraints.

Ensure `.gitignore` contains `*.local.*` to exclude `CLAUDE.local.md`.

Create `.claudeignore` using `examples/.claudeignore` as the starting template; remove sections that don't apply to the project's stack and add any project-specific entries.

## Stack

Goes into `CLAUDE.md` under the `## Stack` section.

## Conventions

Commit style (Conventional Commits when `plan-management` is in use, for its reserved prefixes `plan:`, `chore(ai):`, `Task:`), linting/formatting tools, naming conventions, license. Reflected in `CLAUDE.md`'s Canonical Commands section and informs rule files.

## Rules

Which `.claude/rules/*.md` files to create. A rule file without `paths:` frontmatter auto-loads every session; a rule with `paths:` loads only when a matching file enters context.

```markdown
---
paths:                 # Optional: list of globs. Omit for rules that auto-load every session.
  - <glob>
---

# <Rule title>

## Rules
- <specific, actionable rule>

## Examples
<1–3 short examples.>
```

Reference templates in `examples/`:

| ID | Path | Description | Slots |
|---|---|---|---|
| `git` | `examples/git.md` | Conventional Commits + `plan-management`'s reserved prefixes; git hygiene | type subset, scopes, release tag format, commit ownership |
| `comment` | `examples/comment.md` | Code comment policy; suppresses verbose docstrings | none |
| `claudeignore` | `examples/.claudeignore` | Universal `.claudeignore` starter; excludes build artifacts, deps, generated files, and logs | none |

When a project documentation folder exists (`doc/`, `docs/`, etc.), `CLAUDE.md` carries a one-line pointer to it and `.claude/rules/documentation.md` governs it (path-scoped to `<doc_path>/**`).

## Permissions

What the agent may do autonomously. Goes into `.claude/settings.local.json` (gitignored; `.claude/settings.json` if the user opts for committed permissions).

Eight categories map to settings.json entries:

| Category | What it covers | Maps to |
|---|---|---|
| Bash — allowed | build tools, linters, test runners, package managers | `Bash(npm run *)`, `Bash(git *)` in `allow` |
| Bash — denied | commands that must never run (`rm -rf`, `sudo`, deploy scripts) | `Bash(rm -rf *)`, `Bash(sudo *)` in `deny` |
| File creation | freely, restricted to paths, or confirm first | `Write` / `Edit` / `MultiEdit` in `allow`, optionally path-scoped |
| Protected paths | files/dirs that must not be modified (`.env`, `credentials.*`, prod configs) | `Read(<path>)` and `Edit(<path>)` in `deny` |
| Git commits | auto-commit, push to remote, etc. | `Bash(git commit *)`, `Bash(git push *)` in `allow` / `deny` |
| Network access | `WebSearch` / `WebFetch` during implementation | `WebFetch` / `WebSearch` in `allow` or `deny` |
| Package management | install / upgrade / remove dependencies | relevant patterns in `allow` |
| Agent dispatch | which subagents director may spawn | `Agent(*)` or `Agent(name)` in `allow` |
| Always confirm | operations that always require user approval | omit from all lists — unmatched operations prompt at runtime |

`defaultMode` accepts: `"acceptEdits"` (default), `"default"`, `"dontAsk"`, `"plan"`, `"bypassPermissions"`. Default path is `.claude/settings.local.json` (gitignored).

`ask` lists operations requiring user confirmation. `additionalDirectories` grants access to paths outside the project root. See `examples/settings.json`.

## Layout templates

Named starting points the agent may propose when the user's goal fits. Templates live in `layouts/`; `SKILL.md` carries only the pointer and when-to-consider description.

| Layout | File | When to consider |
|---|---|---|
| Multi-agent | `layouts/multi-agent.md` | Goal decomposes into clearly separate, autonomous responsibilities — each handled by a dedicated agent with its own workflows and memory |

## CLAUDE.md

CLAUDE.md is loaded into every session automatically — it is the model's only persistent context about the project. Write it with that constraint in mind:

- **Target under 200 lines.** Claude Code's built-in system prompt already consumes ~50 instructions; frontier models handle ~150–200 total reliably. Past that, instruction-following degrades uniformly across all instructions.
- **Include only universally applicable content** — every session, every task. Task-specific instructions dilute the file without adding coverage.
- **Pointers, not content.** Reference supplementary files with a one-line description of when to load them. Embedded snippets go stale; file pointers stay accurate.
- **Linters enforce style; CLAUDE.md should not.** Code style rules and schema dumps waste instruction budget on detail Claude won't reliably apply.

The template below is a reference. Sections are customizable — add, remove, or rename based on project requirements.

```markdown
# <Project Name>

<1–3 sentences: what the project does, who it's for, what problem it solves.>

## Key Components
<Omit this section for single-service projects. Otherwise, one sentence per major component explaining its role.>
- `<component>` — <what it does and why it exists>

## Stack
- Language / runtime: <e.g. Python 3.12>
- Framework: <e.g. FastAPI — or "none">
- Key dependencies: <short list with versions>

## Canonical Commands
- Build: `<cmd or "n/a">`
- Test:  `<cmd>`
- Lint:  `<cmd>`
- Run:   `<cmd>`

## Project Documentation
<Omit this section entirely if the project has no doc folder. Otherwise:>

> Project documentation lives at `<doc_path>/`. Do not overwrite human-authored content.
```
