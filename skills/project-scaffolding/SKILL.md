---
name: project-scaffolding
description: Context for setting up or refreshing Claude Code in a project. Covers what information a scaffold requires (goal, stack, conventions, rules, permissions), what files a scaffolded project contains, and their templates. Load before asking to scaffold a project, set a new goal, or be grilled about project setup.
user-invocable: true
---

This skill is the canonical context for Claude Code project setup. It describes what information a scaffold requires, what files result, and what those files look like.

## What a scaffold requires

A complete scaffold is based on five topics:

1. **Goal** — what the agent should achieve in this iteration. Specific enough to decompose into a phase plan: names deliverables, scope boundaries, and known constraints. Goes into `CLAUDE.local.md`.
2. **Stack** — language, runtime, key frameworks, deployment target. Goes into `CLAUDE.md`.
3. **Conventions** — commit style (Conventional Commits when `plan-management` is in use, for its reserved prefixes `plan:`, `chore(ai):`, `Task:`), linting/formatting tools, naming conventions, license.
4. **Rules** — which `.claude/rules/*.md` files to create. Each rule has a one-line scope and either `paths:` frontmatter (loads when a matching file enters context) or none (auto-loads every session).
5. **Permissions** — what the agent may do autonomously. See **Director permissions** below.

## A scaffolded project

A scaffolded project contains:

- `CLAUDE.md` — long-term memory: stack, commands, rules index, project documentation pointer. Committed.
- `CLAUDE.local.md` — short-term memory: current goal and Discovery rationale. Auto-loaded every turn; soft 200-line cap. Gitignored via `*.local.*`.
- `.claude/local/` — spillover for `CLAUDE.local.md` overflow; lowest-importance sections move here, with `CLAUDE.local.md` keeping a 2–3 line summary and pointer. Gitignored explicitly (the `*.local.*` glob doesn't match bare folder names).
- `.claude/rules/*.md` — one rule file per topic.
- `.claude/settings.local.json` — director permissions (gitignored; `.claude/settings.json` if the user opts for committed permissions).
- `.gitignore` — contains `*.local.*` and `.claude/local/`.

When a project documentation folder exists (`doc/`, `docs/`, etc.), `CLAUDE.md` carries a one-line pointer to it and `.claude/rules/documentation.md` governs it (path-scoped to `<doc_path>/**`). The folder itself is never created or modified during scaffolding.

## Requirements Summary

The conversation that produces a scaffold converges on this document. It is approved before any files are written.

````
## Requirements Summary

**Project**: <name — one sentence>
**Description**: <1–3 sentences: what the project does, who it's for, what problem it solves>
**Stack**: <language, runtime, deps, infra>
**Goal**: <what director should achieve in this iteration — 2–4 sentences>
**Out of scope**: <optional — things that might come up but aren't part of this goal>
**Deployment**: <how>
**Testing**: <strategy>
**Team / workflow**: <size, roles, branching, CI>
**Commit convention**: <Conventional Commits, or alternative structured-prefix convention>
**License**: <which and why>
**Project documentation**: <one of: "documented at <doc_path>/" | "opted out">
**Known unknowns**: <open questions about this goal>

**Files to write or overwrite**:
  - `CLAUDE.md` — <create | overwrite>
  - `CLAUDE.local.md` — <create | overwrite>
  - Spillover files (only when CLAUDE.local.md exceeds 200 lines):
    - `.claude/local/<section>.md` — <create | overwrite>
  - Rule files:
    - `.claude/rules/<file>.md` — scope: "<one-line scope>" — <create | overwrite | from reference: <id>>
    - `.claude/rules/documentation.md` — scope: "<doc_path>/**" — create  (only when not opted out)
  - `.claude/settings.local.json` (or `.claude/settings.json`) — <create | overwrite>

**Director permissions** — proposed JSON to be written verbatim:
```json
{
  "permissions": {
    "allow": ["..."],
    "deny": ["..."],
    "defaultMode": "acceptEdits"
  }
}
```

**CLAUDE.local.md** — proposed body to be written verbatim:
```markdown
<rendered CLAUDE.local.md body; if spillover applies, spilled sections are inline summaries pointing at .claude/local/<section>.md, and the spilled bodies are listed below>
```

**Spillover bodies** — only present when CLAUDE.local.md exceeded 200 lines; one block per spilled section:
```markdown
<filename: .claude/local/<section>.md>

<full body for that section>
```

````

## CLAUDE.md template

```markdown
# <Project Name>

<1–3 sentences: what the project does, who it's for, what problem it solves.>

## Stack
- Language / runtime: <e.g. Python 3.12>
- Framework: <e.g. FastAPI — or "none">
- Key dependencies: <short list with versions>

## Canonical Commands
- Build: `<cmd or "n/a">`
- Test:  `<cmd>`
- Lint:  `<cmd>`
- Run:   `<cmd>`

## Rules
- `.claude/rules/<file>.md` — <one-line scope, e.g. "applies to every commit">

## Project Documentation
<Omit this section entirely if the project opted out. Otherwise:>

> Project documentation lives at `<doc_path>/`. Conventions in `.claude/rules/documentation.md` (loads when a doc file enters context). Do not overwrite human-authored content.

## Planning Context
For the current goal and known unknowns, see `CLAUDE.local.md` (auto-loaded by Claude Code; gitignored).
```

## CLAUDE.local.md template

```markdown
# <Project Name> — Current Goal

> Short-term memory for the current goal. Long-term project facts (stack, commands, rules) live in `CLAUDE.md` and `.claude/rules/`. Director's phase/task plan and per-task outcomes live in git history; see the `plan-management` skill. Load the `project-scaffolding` skill to set up a fresh goal for the next iteration.

## Goal
<2–4 sentences describing what director should achieve in this iteration.>

## Out of scope
<Optional — things that might come up but aren't part of this goal. Omit the section if empty.>

## Known Unknowns
<Open questions about this goal that may affect planning.>

## Decisions
<What was chosen during Discovery and why, plus alternatives ruled out. One bullet per decision.>

## Constraints
<Things the goal must respect: deadlines, stakeholder asks, hard limits, things ruled out for non-obvious reasons. One bullet each.>

## Risks & gotchas
<Areas flagged as fragile, surprising, or known to have caused problems. One bullet each with enough context to act on.>

## Decomposition hints
<Suggested phase split or sequencing from Discovery, with rationale. Director may override — these are hints, not mandates.>

## Discovery notes
<Catch-all: stakeholder context, prior attempts, domain vocabulary, anything else director will benefit from that doesn't belong in CLAUDE.md or a rule file.>
```

Write `_None_` as the entire body for any section Discovery produced no material for. Do not omit sections — director relies on the uniform shape.

### Spillover when CLAUDE.local.md exceeds 200 lines

Spill lowest-importance sections one at a time: the section in `CLAUDE.local.md` keeps a 2–3 line summary and a pointer to `.claude/local/<section>.md`; the full body moves to that file.

Spill order (lowest first): `Discovery notes` → `Decomposition hints` → `Risks & gotchas`. The remaining sections (`Goal`, `Out of scope`, `Known Unknowns`, `Decisions`, `Constraints`) stay inline.

Inline summary shape after a section spills:

```markdown
## Discovery notes
<2–3 sentence summary of what's in the spillover file and why director might need it.>
See `.claude/local/discovery-notes.md` for full content.
```

## Rule file template

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

A rule file without `paths:` frontmatter auto-loads every session. A rule with `paths:` loads only when a matching file enters context.

## Director permissions

Eight categories map to settings.json entries:

| Category | What it covers | Maps to |
|---|---|---|
| Bash — allowed | build tools, linters, test runners, package managers that may run freely | `Bash(<prefix>:*)` per command in `allow` |
| Bash — denied | commands that must never run (`rm -rf`, `sudo`, deploy scripts) | `Bash(<prefix>:*)` per command in `deny` |
| File creation | freely, restricted to paths, or confirm first | `Write` / `Edit` / `MultiEdit` in `allow`, optionally path-scoped |
| Protected paths | files/dirs that must not be modified (`.env`, `credentials.*`, prod configs) | `Read(<path>)` and `Edit(<path>)` in `deny` |
| Git commits | auto-commit, push to remote, etc. | relevant `Bash(git ...)` patterns in `allow` / `deny` |
| Network access | `WebSearch` / `WebFetch` during implementation | `WebFetch` / `WebSearch` in `allow` or `deny` |
| Package management | install / upgrade / remove dependencies | relevant patterns in `allow` |
| Always confirm | operations that always require user approval | omit from both lists — unmatched operations prompt at runtime |

Default `defaultMode` is `"acceptEdits"`. Default path is `.claude/settings.local.json` (gitignored).

### settings.json schema

```json
{
  "permissions": {
    "allow": ["Bash(npm test:*)", "Bash(npm run lint:*)", "Read", "Edit"],
    "deny":  ["Bash(rm -rf:*)", "Bash(sudo:*)", "Read(./.env)"],
    "defaultMode": "acceptEdits"
  }
}
```

## Reference templates

The `references/` directory ships hybrid starter templates for recurring rule files: universal content verbatim, plus labelled slots filled from Discovery answers.

| ID | Path | Description | Slots filled from |
|---|---|---|---|
| `git` | `references/git.md` | Conventional Commits + `plan-management`'s reserved prefixes; universal git hygiene | Discovery — type subset, scopes, release tag format, commit ownership |
| `comment` | `references/comment.md` | Code comment policy; suppresses verbose docstrings and phase-decision narration | None — no slots |
