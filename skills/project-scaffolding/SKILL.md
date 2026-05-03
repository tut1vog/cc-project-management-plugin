---
name: project-scaffolding
description: Read the canonical layout for Claude Code project scaffolding (CLAUDE.md, CLAUDE.local.md, rule files, settings.local.json) and (for scaffolder only) write that scaffolding from an approved Requirements Summary. Use when you need to know the structure of a scaffolded project's instruction files, the overwrite/git-bootstrap policy, or the Requirements Summary input contract. Triggers on "project scaffolding", "CLAUDE.md template", "rule file template", "director permissions schema", "requirements summary contract".
user-invocable: false
---

This skill is the canonical spec for Claude Code project scaffolding — the files scaffolder writes after Discovery so director and other agents have a stable starting point.

## Who writes / who reads

- **Scaffolder writes** — only it executes the scaffold procedure, and only after the user approves a Requirements Summary.
- **Anyone reads** — any agent or rule may consult this skill for templates, schemas, and contracts.

## Procedure preconditions (hard gate)

Before executing any step in the **Scaffold procedure** below, confirm:

1. You are scaffolder.
2. The user has explicitly approved a Requirements Summary in the current session, in the shape defined by **Requirements Summary input contract**.

If either is false, **stop** and direct the user to invoke scaffolder — the procedure overwrites files without prompting and is not safe to fire from any other context.

## File-system mental model

- `CLAUDE.md` — **long-term memory**: stack, commands, rules index, project documentation pointer. Committed.
- `CLAUDE.local.md` — **short-term memory**: current goal + Discovery rationale (decisions, constraints, risks, decomposition hints, Discovery notes). Auto-loaded every turn; soft 200-line cap. Gitignored via `*.local.*`. Disposable — re-run scaffolder for the next goal.
- `.claude/local/` — **spillover** for `CLAUDE.local.md` overflow; lowest-importance sections move here and `CLAUDE.local.md` keeps a 2–3 line summary + pointer. Director reads on demand. Gitignored explicitly (the `*.local.*` glob doesn't match bare folder names). See **Spillover when CLAUDE.local.md exceeds 200 lines** below.
- `<doc_path>/` (project documentation folder when present — `doc/`, `docs/`, etc.) — durable, project-describing content. Located via the Project Documentation pointer in `CLAUDE.md`. Never written at scaffold time.

## Project documentation folder

The Project Documentation section in `CLAUDE.md` takes one of three shapes (or is omitted entirely if the project opts out):

| Probe result at scaffold time | Shape |
|---|---|
| Doc folder exists, convention file detected | **A** — pointer to existing folder + convention file |
| Doc folder exists, no convention file (folder may have content or be empty) | **A** (default; folder content read ad hoc) — or **B** / **C** if the user opts in to capturing conventions |
| No doc folder | **B** / **C** / opt-out per Discovery |

Threshold for B vs C:

- Conventions ≤10 lines of markdown → **Shape B**: inline in `CLAUDE.md`'s Project Documentation section.
- Conventions >10 lines → **Shape C**: write `.claude/rules/documentation.md` with `paths: [<doc_path>/**]` frontmatter so it loads only when a doc file enters context; `CLAUDE.md` keeps the one-line pointer.

The `<doc_path>/` folder itself is never touched at scaffold time — no files are created or overwritten there.

## Requirements Summary input contract

Scaffolder's Discovery produces this summary at the end of its interview phases. The user must approve it verbatim before the Scaffold procedure runs.

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
**Project documentation**: <one of: "<doc_path>/ — Shape A (existing, convention file: <path>)" | "<doc_path>/ — Shape A (existing, no convention file)" | "<doc_path>/ — Shape B (new, inline conventions)" | "<doc_path>/ — Shape C (new, conventions in rule file)" | "opted out">
**Known unknowns**: <open questions about this goal>

**Files I will write or overwrite in Scaffold**:
  - `CLAUDE.md` — <create | overwrite>
  - `CLAUDE.local.md` — <create | overwrite>
  - Spillover files (only listed when CLAUDE.local.md exceeds 200 lines and a section spills):
    - `.claude/local/<section>.md` — <create | overwrite>
  - Rule files:
    - `.claude/rules/<file>.md` — scope: "<one-line scope>" — <create | overwrite | from reference: <id>>
    - <repeat for each>
    - `.claude/rules/documentation.md` — scope: "<doc_path>/**" — create  (only when Project documentation Shape C)
  - `.claude/settings.local.json` (or `.claude/settings.json` if user opted for committed permissions) — <create | overwrite>

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
<rendered CLAUDE.local.md body matching the CLAUDE.local.md template; if spillover applies, the spilled sections are inline summaries pointing at .claude/local/<section>.md, and the spilled bodies are listed below>
```

**Spillover bodies** — only present when CLAUDE.local.md exceeded 200 lines; one block per spilled section:
```markdown
<filename: .claude/local/<section>.md>

<full body for that section>
```

**Project documentation conventions** — only present for Shape B and C; the conventions to be written verbatim into `CLAUDE.md`'s Project Documentation section (Shape B) or into `.claude/rules/documentation.md` (Shape C):
```markdown
<bullets or prose capturing the project's documentation conventions>
```
````

## Scaffold procedure

### Step 1 — Ensure a git repository exists

Run `git rev-parse --is-inside-work-tree` in the project root.

- **Not a git repo**: run `git init`. If the tree has any files, `git add -A && git commit -m "chore: initial snapshot before Claude Code scaffolding"`.
- **Git repo, dirty tree**: stop and ask the user to commit, stash, or abort. Only proceed after an explicit choice — that's what makes the overwrite-without-prompt policy safe.
- **Git repo, clean tree**: proceed.

Then ensure `.gitignore` contains both `*.local.*` and `.claude/local/` (the latter is needed because `*.local.*` doesn't match bare folder names):

```bash
grep -qxF '*.local.*'   .gitignore 2>/dev/null || echo '*.local.*'   >> .gitignore
grep -qxF '.claude/local/' .gitignore 2>/dev/null || echo '.claude/local/' >> .gitignore
```

### Step 2 — Write `CLAUDE.md`

Use the template in **CLAUDE.md template** below. Populate every section from the approved Requirements Summary. Overwrite any existing file.

### Step 3 — Write `CLAUDE.local.md` and any spillover

Write `CLAUDE.local.md` from the **CLAUDE.local.md** block in the Requirements Summary. If the summary includes **Spillover bodies**, also create `.claude/local/` and write each spilled section to its `.claude/local/<section>.md` path verbatim. Overwrite existing files.

Verify `wc -l CLAUDE.local.md` ≤ 200; spill more if it overflows.

### Step 4 — Write `.claude/rules/*.md`

Create the `.claude/rules/` directory if missing. For every rule file listed in the approved Requirements Summary:

- `from reference: <id>` entries: read `references/<id>.md` and fill its labelled slots from Discovery answers.
- Other entries: use the **Rule file template** below, filled from Discovery answers.

Each rule file must stand on its own (one screen, self-contained). Overwrite any existing file. Don't manufacture rules the user didn't specify.

### Step 5 — Write `.claude/settings.local.json`

Write the **Director permissions** JSON block verbatim to `.claude/settings.local.json` (or `.claude/settings.json` if the user opted in during Discovery). Validate:

```bash
python3 -m json.tool .claude/settings.local.json > /dev/null
```

### Step 6 — Print the Scaffold Summary

Print a list of every file created or overwritten:

```
## Scaffold Summary
Written / overwritten:
- CLAUDE.md
- CLAUDE.local.md
- .claude/local/<section>.md       (only when spillover applied)
- ...
- .claude/rules/<file>.md
- ...
- .claude/settings.local.json

Verify with: git status && git diff --stat
(Gitignored files won't appear in git diff.)
```

## CLAUDE.md template

Write exactly this structure:

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
<Section content depends on what was discovered or decided in Discovery — see **Project documentation folder** in this skill. Three shapes; pick the one that applies, or omit the section entirely if the project opted out of director-maintained documentation.

Shape A — existing doc folder (with or without convention file):
> Project documentation lives at `<doc_path>/`. Before writing or editing files there, read existing files (especially `<convention_file>` if present) to match the project's existing structure, naming, and style. Do not overwrite human-authored content.

Shape B — new project, conventions short enough to inline (≤10 lines):
> Project documentation lives at `<doc_path>/`. Conventions:
> - <bullet>
> - <bullet>
> Do not overwrite human-authored content.

Shape C — new project, conventions too long to inline (>10 lines):
> Project documentation lives at `<doc_path>/`. Conventions documented in `.claude/rules/documentation.md` (path-scoped to `<doc_path>/**`). Do not overwrite human-authored content.>

## Planning Context
For the current goal and known unknowns, see `CLAUDE.local.md` (auto-loaded by Claude Code; gitignored).
```

## CLAUDE.local.md template

Write exactly this structure:

```markdown
# <Project Name> — Current Goal

> Short-term memory for the current goal. Long-term project facts (stack, commands, rules) live in `CLAUDE.md` and `.claude/rules/`. Director's phase/task plan and per-task outcomes live in git history; see the `plan-management` skill. When this goal is met, re-run scaffolder to dispose this file and scaffold a fresh one for the next goal.

## Goal
<2–4 sentences describing what director should achieve in this iteration.>

## Out of scope
<Optional — things that might come up but aren't part of this goal. Omit the section if there's nothing to list.>

## Known Unknowns
<Open questions about this goal that may affect planning.>

## Decisions
<What was chosen during Discovery and why, plus alternatives ruled out and why. One bullet per decision.>

## Constraints
<Things the user explicitly said the goal must respect: deadlines, stakeholder asks, hard limits, things ruled out of scope for non-obvious reasons. One bullet each.>

## Risks & gotchas
<Areas of the codebase or problem space the user flagged as fragile, surprising, or known to have bitten them. One bullet each, with enough context that director can act on it.>

## Decomposition hints
<Suggested phase split or sequencing the user surfaced during Discovery, with rationale. Director may override these — they are hints, not mandates.>

## Discovery notes
<Catch-all: stakeholder context, prior attempts and why they were abandoned, domain vocabulary, anything else from Discovery director will benefit from that does not belong in CLAUDE.md / a rule file.>
```

Write `_None_` as the entire section body for any section the Discovery did not produce material for. Do not omit sections — director relies on the uniform shape.

### Spillover when CLAUDE.local.md exceeds 200 lines

When the populated body exceeds 200 lines, spill lowest-importance sections one at a time: the section in `CLAUDE.local.md` keeps a 2–3 line summary + pointer to `.claude/local/<section>.md`, and the full body moves to that file.

Spill order (lowest first): `Discovery notes` → `Decomposition hints` → `Risks & gotchas`. The remaining sections (`Goal`, `Out of scope`, `Known Unknowns`, `Decisions`, `Constraints`) are highest-importance and stay inline. Spill until `wc -l CLAUDE.local.md` ≤ 200.

Inline summary shape after a section spills:

```markdown
## Discovery notes
<2–3 sentence summary capturing what's in the spillover file and why director might need it.>
See `.claude/local/discovery-notes.md` for full content.
```

The spillover file holds the full section body — same heading and bullet structure as the original.

## Rule file template

```markdown
---
paths:                 # Optional: list of globs scoping when the rule loads. Omit for rules that should auto-load every session.
  - <glob>
  - <glob>
---

# <Rule title>

## Rules
- <specific, actionable rule>
- <specific, actionable rule>

## Examples
<1–3 short examples — a good commit message, a valid test file name, an acceptable docs change, etc.>
```

Pull concrete content from Phase 0 findings and Discovery answers. Each rule must stand alone — a reader who opens only that file must know what to do.

A rule file without `paths:` frontmatter auto-loads every session (use for broadly-applicable rules like commit hygiene). A rule with `paths:` loads only when a matching file enters context (use for narrowly-scoped rules so they don't pay an always-loaded token cost).

## Director permissions interview structure

Discovery walks the user through these eight categories one at a time, proposing defaults from the agreed stack. Each row maps to entries in the **settings.json schema** below.

| Category | Prompt the user on | Maps to |
|---|---|---|
| Bash — allowed | build tools, linters, test runners, package managers that may run freely | `Bash(<prefix>:*)` per command in `allow` |
| Bash — denied | commands that must never run (`rm -rf`, `sudo`, deploy scripts) | `Bash(<prefix>:*)` per command in `deny` |
| File creation | freely, restricted to paths, or confirm first | `Write` / `Edit` / `MultiEdit` (bare or scoped with `Write(<path>/**)`) in `allow` |
| Protected paths | files/dirs that must not be modified (`.env`, `credentials.*`, prod configs) | `Read(<path>)` and `Edit(<path>)` in `deny` |
| Git commits | auto-commit, push to remote, etc. | relevant `Bash(git ...)` patterns in `allow` / `deny` |
| Network access | `WebSearch` / `WebFetch` during implementation; forbidden APIs | `WebFetch` / `WebSearch` in `allow` or `deny` |
| Package management | install / upgrade / remove dependencies | relevant patterns in `allow` (e.g. `Bash(pip install:*)`, `Bash(npm install:*)`) |
| Always confirm | operations that always require user approval (delete files, drop tables, force-push) | **Omit from both lists.** Unmatched operations prompt at runtime. |

After the interview, render the result as the JSON shown in the **settings.json schema** below into the Requirements Summary's **Director permissions** block. Default `defaultMode` to `"acceptEdits"`. Default path is `.claude/settings.local.json` (gitignored); confirm before writing to `.claude/settings.json` instead.

## settings.json schema

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

The `references/` directory ships hybrid starter templates for recurring rules: universal content verbatim + labelled slots filled from Discovery answers.

| ID | Path | Description | Slots filled from |
|---|---|---|---|
| `git` | `references/git.md` | Conventional Commits + `plan-management`'s reserved prefixes (`plan:`, `chore(ai):`, `Task:`); universal git hygiene rules | Phase 5 — type subset, custom scopes, release tag format, commit ownership |

Phase 6 marks reference entries as `from reference: <id>`; Step 4 fills their slots via `Edit` calls.

## Out of scope

This skill writes **rule** templates only. Agent authoring (`.claude/agents/*.md`) is director's responsibility — see `agents/director.md`.
