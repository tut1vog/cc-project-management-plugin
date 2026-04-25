# Plan

## Status
Current phase: Phase 3 — Replace `project-brief.md` with `CLAUDE.local.md`
Current task: 3.1 — Refactor `initializer`/`advisor` agents and the agent-authoring rule

---

## Phases

### Phase 3: Replace `project-brief.md` with `CLAUDE.local.md`
> Drop the custom `project-brief.md` handoff in favour of two Claude Code-native files: `CLAUDE.md` (committed; long-term rules) and `CLAUDE.local.md` (gitignored; current goal). Both are auto-loaded by Claude Code, so director gets planning context without an explicit read. Detailed design in `/home/ubun/.claude/plans/currently-initializer-and-advisor-velvety-pie.md`.

| ID | Task | Status |
|----|------|--------|
| 3.1 | Refactor `agents/initializer.md`, `agents/advisor.md`, `.claude/rules/agent-authoring.md` | in-progress |
| 3.2 | Update `docs/*.md` and `README.md` to reflect the new flow | pending |
| 3.3 | Bump `.claude-plugin/plugin.json` version to `0.3.0` | pending |

---

## Current Task

**ID**: 3.1
**Title**: Refactor `agents/initializer.md`, `agents/advisor.md`, `.claude/rules/agent-authoring.md`
**Phase**: Phase 3 — Replace `project-brief.md` with `CLAUDE.local.md`
**Status**: in-progress

### Goal
Stop scaffolding `project-brief.md`. Both initializer and advisor now scaffold a `CLAUDE.local.md` file alongside `CLAUDE.md`, with content split along a clean axis: `CLAUDE.md` holds long-term rules (stack, layout, commands, constraints, rules index); `CLAUDE.local.md` holds the current goal (project overview, scope, director permissions, known unknowns). The agent-authoring rule that defines this contract is updated in the same change so the agents and the rule stay in sync.

### Context

**Files to modify** (paths relative to repo root `/home/ubun/github.com/tut1vog/cc-project-management-plugin/`):
- `agents/initializer.md`
- `agents/advisor.md`
- `.claude/rules/agent-authoring.md`

**Files NOT to modify** (deferred per user choice — agents-only scope):
- `/CLAUDE.md` (this repo's own)
- `/project-brief.md` (this repo's own)
- `/.gitignore` (this repo's own)

**Current state of the templates** (from prior exploration):
- `agents/initializer.md` lines 117–149 — current `CLAUDE.md` template (no `## Constraints`, ends with `## Planning Context` pointing to `project-brief.md`).
- `agents/initializer.md` lines 220–248 — current Mode 3 Handoff Step 1 that writes `project-brief.md`. To be deleted entirely.
- `agents/advisor.md` lines 169–201 — current `CLAUDE.md` template (identical to initializer's).
- `agents/advisor.md` lines 273–304 — current Mode 3 Handoff Step 1 that writes `project-brief.md` (with extra `Current State` section, Stable/Planned scope variant). To be deleted.
- `agents/advisor.md` Mode 0 Audit ("What to read", line ~40) — currently lists `CLAUDE.md` etc. but not `project-brief.md`. Needs to add `CLAUDE.local.md` and any legacy `project-brief.md` so its content can be migrated.
- `agents/advisor.md` Mode 0 Audit Summary table (line ~61) — currently has a `CLAUDE.md` row. Add a `CLAUDE.local.md` row.
- Both agents have a Phase 7 description (initializer line 56, advisor line 108) referring to `project-brief.md` — change to `CLAUDE.local.md`.
- Both agents' Requirements Summary (initializer lines 74–97, advisor lines 126–149) list `CLAUDE.md` sections — add `Constraints` to that list and add a separate `CLAUDE.local.md` line.
- `.claude/rules/agent-authoring.md` (line ~12) has the rule "advisor and initializer both produce `project-brief.md` for `director` to consume" — rewrite to `CLAUDE.local.md`. Same file lists shared blocks at the bottom — replace "Handoff template" with "CLAUDE.local.md template".

**Templates** (use these verbatim — they were locked in during planning):

`CLAUDE.md` — durable, **identical** in both agents:
```markdown
# <Project Name>

<One-sentence purpose — what the project does and for whom.>

## Stack
- Language / runtime: <e.g. Python 3.12>
- Framework: <e.g. Typer, FastAPI, React — or "none">
- Key dependencies: <short list with versions>

## Directory Layout
<Top-level dirs and what each holds. Keep to 5–15 lines.>

## Canonical Commands
- Build: `<cmd or "n/a">`
- Test:  `<cmd>`
- Lint:  `<cmd>`
- Run:   `<cmd>`

## Constraints
<Language, platform, team, compliance, budget — everything that limits choices. Long-term limits, not transient scope.>

## Rules (load on demand)
Each rule file below is a focused behavioral contract. Read a rule file when its trigger matches your task — do not auto-load.

- `.claude/rules/<file>.md` — <one-line trigger, e.g. "read before making any commit">

## Planning Context
For current goal, scope, director permissions, and known unknowns, see `CLAUDE.local.md` (auto-loaded by Claude Code; gitignored).
```

`CLAUDE.local.md` — initializer variant (no Current State, MVP/Deferred):
```markdown
# <Project Name> — Current Goal

> Long-term project facts (stack, commands, rules, constraints) live in `CLAUDE.md` and `.claude/rules/`. This file is the planning input for director: what we're building right now and what the director may do autonomously.

## Project Overview
<What the project is, who it's for, what problem it solves.>

## Scope
**MVP**: <what's in>
**Deferred**: <what's out>

## Director Permissions
<Filled-in Phase 7 Director Permissions table verbatim. Same permissions are encoded in `.claude/settings.json` for machine enforcement; this is the human-readable copy director consults while planning.>

## Known Unknowns
<Open questions that may affect planning.>
```

`CLAUDE.local.md` — advisor variant (adds Current State; Stable/Planned):
```markdown
# <Project Name> — Current Goal

> Long-term project facts (stack, commands, rules, constraints) live in `CLAUDE.md` and `.claude/rules/`. This file is the planning input for director: what's already stable, what's planned, and what the director may do autonomously.

## Project Overview
<What the project is, who it's for, what problem it solves.>

## Current State
<Summary of what exists from the audit: stack, notable subsystems, existing CI, existing Claude Code setup before this run.>

## Scope
**Stable**: <what's already built and working>
**Planned**: <what's still to be done>

## Director Permissions
<Filled-in Phase 7 Director Permissions table verbatim. Same permissions are encoded in `.claude/settings.json` for machine enforcement; this is the human-readable copy director consults while planning.>

## Known Unknowns
<Open questions that may affect planning.>
```

**Director Orient**: no change. Claude Code auto-loads both `CLAUDE.md` and `CLAUDE.local.md` at session start; director needs no explicit read step. `agents/director.md` itself contains zero references to either filename and is not modified.

### Implementation Steps

1. **`agents/initializer.md` edits**:
   - **Phase 7 description** (line ~56): replace "later referenced verbatim by the Requirements Summary and `project-brief.md`" → "… and `CLAUDE.local.md`".
   - **Requirements Summary** (lines ~74–97): in the Claude Code setup checklist, add `Constraints` to the `CLAUDE.md` sections list and add a new `CLAUDE.local.md` line listing its sections (Project Overview, Scope, Director Permissions, Known Unknowns). Remove any reference to `project-brief.md`.
   - **Mode 2 Scaffold intro** (line ~106): rewrite the "everything you write here is persistent — `project-brief.md` (written next in Handoff) is the planning input" sentence. New framing: both `CLAUDE.md` (long-term rules) and `CLAUDE.local.md` (current goal) are written in Scaffold; there is no separate Handoff write.
   - **Scaffold Step 1 (git repo)**: extend to also ensure `.gitignore` exists at the project root and contains a line `CLAUDE.local.md`. Behaviour: if `.gitignore` is missing, create it with that one line; if present and the line is absent, append it; if present and the line is already there, no-op.
   - **Scaffold Step 2 — Write CLAUDE.md** (line ~117): replace the template with the new one above (adds `## Constraints`; updates final `## Planning Context` pointer to `CLAUDE.local.md`).
   - **Scaffold Step 2.5 (NEW) — Write CLAUDE.local.md**: insert a new sub-step using the initializer variant template above (no `Current State`).
   - **Scaffold Summary** (line ~210): list `CLAUDE.local.md` in addition to `CLAUDE.md`, `.claude/rules/*`, `.claude/settings.json`.
   - **Mode 3 Handoff** (lines ~220–258): delete Step 1 (the project-brief.md write block — including the template at lines 228–248). Mode 3 becomes a short post-Scaffold message: tell the user what was scaffolded and that they can now invoke director.
   - **Final user instruction list** (line ~256): replace `project-brief.md (planning input)` with `CLAUDE.local.md (current goal — gitignored)`.

2. **`agents/advisor.md` edits** — same as initializer plus advisor-only:
   - **Mode 0 Audit "What to read"** (line ~40): add reading existing `CLAUDE.local.md` if present, and any legacy `project-brief.md` (so the advisor can carry forward its scope/permissions/unknowns into the rewritten `CLAUDE.local.md`).
   - **Mode 0 Audit Summary table** (line ~61): add a `CLAUDE.local.md` row (alongside the existing `CLAUDE.md` row). If a legacy `project-brief.md` exists, add a row noting it as legacy with intent to migrate.
   - **Phase 7 description** (line ~108): same `project-brief.md` → `CLAUDE.local.md` swap.
   - **Requirements Summary** (lines ~126–149): same as initializer — add `Constraints` to `CLAUDE.md` and add `CLAUDE.local.md` line. Keep advisor-only `Current Claude Code setup` field.
   - **Scaffold intro and steps** (line ~158 onwards): mirror the initializer changes (Scaffold Step 1 .gitignore, Scaffold Step 2 CLAUDE.md template, Scaffold Step 2.5 CLAUDE.local.md template using the **advisor variant** with `Current State` and Stable/Planned scope).
   - **Scaffold overwrite policy** (line ~160): replace "`project-brief.md`" with "`CLAUDE.local.md`". Add explicit note that any legacy `project-brief.md` is **left in place** (not auto-deleted) — its content is migrated into the new `CLAUDE.local.md` in this scaffold pass.
   - **Mode 3 Handoff** (lines ~273–314): delete Step 1's project-brief.md write block. Replace with the same short post-Scaffold message used in initializer.
   - **Final user instruction list** (line ~312): replace `project-brief.md (planning input)` with `CLAUDE.local.md (current goal — gitignored)`.

3. **`.claude/rules/agent-authoring.md` edits**:
   - **Contract sentence** (line ~12): rewrite "advisor and initializer both produce `project-brief.md` for `director` to consume" → "advisor and initializer both produce `CLAUDE.local.md` for `director` to consume (auto-loaded by Claude Code at session start; gitignored)".
   - **Shared-blocks list** at the bottom of the Rules section: replace "Handoff template" (no longer exists — Handoff no longer writes a file) with "CLAUDE.local.md template (in Scaffold)". Keep all other shared-block entries.
   - **Examples** that show project-brief.md (around lines 35, 38): leave as-is — those examples illustrate project-agnostic vs. project-specific reference patterns and the filename change doesn't break the lesson. (Optional small edit: swap to `CLAUDE.local.md` in the examples to keep the rule self-consistent.)

### Verification

Run from repo root `/home/ubun/github.com/tut1vog/cc-project-management-plugin/`:

- [ ] `python3 -m json.tool .claude-plugin/plugin.json` — JSON valid (no version change yet at this task; that lands in 3.3).
- [ ] `python3 -m json.tool .claude-plugin/marketplace.json` — JSON valid.
- [ ] `python3 -m json.tool .mcp.json` — JSON valid.
- [ ] `claude --plugin-dir ./` then `/agents` — all three agents load (no YAML frontmatter regressions).
- [ ] `grep -rn 'project-brief\.md' agents/ .claude/rules/` — only hits are in `agents/advisor.md` (Mode 0 Audit reads legacy `project-brief.md`; Scaffold overwrite policy notes legacy file is left in place). Zero hits in `agents/initializer.md`. Zero hits in `agents/director.md` (already clean). At most a stylistic example hit in `.claude/rules/agent-authoring.md` if examples are left as-is.
- [ ] `grep -rn 'CLAUDE\.local\.md' agents/ .claude/rules/` — hits in: both agents (Phase 7, Requirements Summary, Scaffold intro, Scaffold Step 1 .gitignore, Scaffold Step 2.5 write, Scaffold Summary, post-Scaffold user message) and `.claude/rules/agent-authoring.md` (contract sentence, shared-blocks list).
- [ ] Read `agents/initializer.md` end-to-end and confirm: (a) `## Constraints` appears in the CLAUDE.md template between `## Canonical Commands` and `## Rules (load on demand)`; (b) a Scaffold Step 2.5 writes `CLAUDE.local.md` with no `## Current State`; (c) Mode 3 no longer writes any file; (d) Scaffold Step 1 ensures `.gitignore` lists `CLAUDE.local.md`.
- [ ] Read `agents/advisor.md` end-to-end and confirm the same plus: (e) Scaffold Step 2.5 advisor variant template includes `## Current State` and uses Stable/Planned scope; (f) Mode 0 Audit reads any legacy `project-brief.md` for migration; (g) overwrite policy notes legacy `project-brief.md` is not deleted.
- [ ] Read `.claude/rules/agent-authoring.md` and confirm the contract sentence and shared-blocks list are updated.

### Suggested Agent
`general-purpose` — task is large, requires careful coordinated edits across three Markdown files with shared sections. No specialised subagent better fits this kind of synchronised refactor.
