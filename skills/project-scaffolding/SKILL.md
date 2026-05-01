---
name: project-scaffolding
description: Read the canonical layout for Claude Code project scaffolding (CLAUDE.md, CLAUDE.local.md, rule files, settings.local.json) and (for scaffolder only) write that scaffolding from an approved Requirements Summary. Use when you need to know the structure of a scaffolded project's instruction files, the overwrite/git-bootstrap policy, or the Requirements Summary input contract. Triggers on "project scaffolding", "CLAUDE.md template", "rule file template", "director permissions schema", "requirements summary contract".
---

This skill is the canonical spec for Claude Code project scaffolding — the durable files scaffolder writes after Discovery so director and other agents have a stable starting point. Templates, interview schemas, and the write procedure all live here; agents and humans should treat this as the single source of truth.

## Who writes / who reads

- **Scaffolder writes.** Scaffolder is the only agent that executes the scaffold procedure, and only after the user has explicitly approved a Requirements Summary in scaffolder's Discovery mode.
- **Anyone reads.** Any agent or rule may consult this skill for the file templates, the overwrite/git-bootstrap policy, the Director permissions interview structure, or the Requirements Summary input contract.

## Procedure preconditions (hard gate)

Before executing any step in the **Scaffold procedure** below, confirm:

1. You are scaffolder.
2. The user has explicitly approved a Requirements Summary in the current session, in the shape defined by **Requirements Summary input contract**.

If either is false, **stop**. Direct the user to invoke scaffolder; the procedure overwrites `CLAUDE.md` and rule files without prompting and is not safe to fire from any other context.

## File-system mental model

`CLAUDE.md` is **long-term memory** for the project — facts that survive across goals (stack, commands, rules index). It is committed and maintained.

`CLAUDE.local.md` is **short-term memory** — the current goal director is working toward. Disposable: when a goal is met, re-run scaffolder to scaffold a fresh `CLAUDE.local.md` for the next goal. Gitignored via `*.local.*`.

Both files are auto-loaded by Claude Code at session start.

## Requirements Summary input contract

Scaffolder's Discovery produces this summary at the end of its interview phases. The user must approve it verbatim before the Scaffold procedure runs. Render the summary using exactly this structure:

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
**Known unknowns**: <open questions about this goal>

**Files I will write or overwrite in Scaffold**:
  - `CLAUDE.md` — <create | overwrite>
  - `CLAUDE.local.md` — <create | overwrite>
  - Rule files:
    - `.claude/rules/<file>.md` — trigger: "<one-line trigger>" — <create | overwrite | from reference: <id>>
    - <repeat for each>
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
````

The **Director permissions** block is the literal JSON the procedure will write to disk; the user approves the JSON form directly, with no separate human-readable rendering.

## Scaffold procedure

### Step 1 — Ensure a git repository exists

Run `git rev-parse --is-inside-work-tree` in the project root.

- **Not a git repo**: run `git init`. If the tree has any files, `git add -A && git commit -m "chore: initial snapshot before Claude Code scaffolding"`.
- **Git repo, dirty tree**: stop and ask the user to (a) commit as a snapshot, (b) stash, or (c) abort. Only proceed after an explicit choice — this is what makes the overwrite-without-prompt policy safe; users recover prior content via `git diff` / `git checkout`.
- **Git repo, clean tree**: proceed.

After git state is settled, ensure `.gitignore` at the project root contains the single line `*.local.*`. This one glob covers `CLAUDE.local.md`, `settings.local.json`, and any other `*.local.*` Claude Code conventions.

```bash
grep -qxF '*.local.*' .gitignore 2>/dev/null || echo '*.local.*' >> .gitignore
```

### Step 2 — Write `CLAUDE.md`

Use the template in **CLAUDE.md template** below. Populate every section from the approved Requirements Summary. Overwrite any existing file without prompting (Step 1's git bootstrap makes recovery possible).

### Step 3 — Write `CLAUDE.local.md`

Use the template in **CLAUDE.local.md template** below. Populate from the approved Requirements Summary. Overwrite any existing file.

### Step 4 — Write `.claude/rules/*.md`

Create the `.claude/rules/` directory if missing. For every rule file listed in the approved Requirements Summary:

- If the entry references a reference template by id (e.g. `from reference: git`), read `references/<id>.md` from this skill and use it as the starting point. Universal-truth rules in the template stay verbatim; fill the labelled slots with project-specific content from Discovery answers.
- Otherwise, use the **Rule file template** below as the structure and fill in project-specific content from Discovery answers.

Each rule file must stand on its own (one screen, self-contained). Overwrite any existing file. Do not manufacture rules the user did not specify; generic fluff ("write clear code") does not belong in a rule file.

### Step 5 — Write `.claude/settings.local.json`

Write the **Director permissions** JSON block from the Requirements Summary verbatim to the file at the chosen path: `.claude/settings.local.json` by default; `.claude/settings.json` only if the user explicitly opted for committed permissions during Discovery. Overwrite any existing file.

Validate after writing:

```bash
python3 -m json.tool .claude/settings.local.json > /dev/null
```

Fix any error before continuing.

### Step 6 — Print the Scaffold Summary

Print a list of every file created or overwritten:

```
## Scaffold Summary
Written / overwritten:
- CLAUDE.md
- CLAUDE.local.md
- .claude/rules/<file>.md
- ...
- .claude/settings.local.json

Verify with: git status && git diff --stat
(Note: CLAUDE.local.md and .claude/settings.local.json are gitignored and won't appear in git diff.)
```

## CLAUDE.md template

`CLAUDE.md` is long-term memory: facts that survive across goals. Write exactly this structure:

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

## Rules (load on demand)
Each rule file below is a focused behavioral contract. Read a rule file when its trigger matches your task — do not auto-load.

- `.claude/rules/<file>.md` — <one-line trigger, e.g. "read before making any commit">

## Planning Context
For the current goal and known unknowns, see `CLAUDE.local.md` (auto-loaded by Claude Code; gitignored).
```

The filesystem is the source of truth for directory structure; `tree` / `Glob` reveals it on demand. Do not maintain a directory tree in `CLAUDE.md`. If a specific path has a non-obvious operationally load-bearing role (e.g. "`bin/` is on PATH while the plugin is enabled"), capture it as a focused rule file rather than padding `CLAUDE.md`.

Operational constraints (compliance, performance/SLA targets, compatibility commitments) belong in dedicated rule files with their own triggers, not a generic `Constraints` section.

## CLAUDE.local.md template

`CLAUDE.local.md` is short-term memory: disposable when the current goal is met. Write exactly this structure:

```markdown
# <Project Name> — Current Goal

> Short-term memory for the current goal. Long-term project facts (stack, commands, rules) live in `CLAUDE.md` and `.claude/rules/`. Director's phase/task plan and per-task outcomes live in git history; see the `plan-management` skill. When this goal is met, re-run scaffolder to dispose this file and scaffold a fresh one for the next goal.

## Goal
<2–4 sentences describing what director should achieve in this iteration.>

## Out of scope
<Optional — things that might come up but aren't part of this goal. Omit the section if there's nothing to list.>

## Known Unknowns
<Open questions about this goal that may affect planning.>
```

## Rule file template

```markdown
# <Rule title>

## Rules
- <specific, actionable rule>
- <specific, actionable rule>

## Examples
<1–3 short examples — a good commit message, a valid test file name, an acceptable docs change, etc.>
```

Pull concrete content from Phase 0 findings (existing linter configs, CI workflow commands, commit history patterns) and Discovery answers. The rule must stand on its own — a reader who opens only that file must know exactly what to do.

## Director permissions interview structure

Discovery's permissions interview walks the user through these eight categories, one at a time, proposing defaults from the agreed stack. Each row maps directly to entries in `.claude/settings.local.json` per the **settings.json schema** below.

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

After walking the eight categories, render the result as the literal `.claude/settings.local.json` JSON for the Requirements Summary's **Director permissions** block. Default `defaultMode` to `"acceptEdits"` unless the user asked for stricter oversight.

By default, permissions are written to `.claude/settings.local.json` (gitignored, per-developer). Confirm with the user before writing them to `.claude/settings.json` instead.

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

Any operation not listed in `allow` or `deny` is prompted at runtime — that is how the **Always confirm** category from the interview is encoded (by omission).

## Reference templates

The `references/` directory ships starter templates for rules that recur across projects scaffolded with this plugin. Each template is **hybrid**: universal-truth rules included verbatim, plus labelled slots the scaffolder fills from Discovery answers.

| ID | Path | Description | Slots filled from |
|---|---|---|---|
| `git` | `references/git.md` | Conventional Commits + `plan-management`'s reserved prefixes (`plan:`, `chore(ai):`, `Task:`); universal git hygiene rules | Phase 5 — type subset, custom scopes, release tag format, commit ownership |

When scaffolder proposes a rule from this catalog during Phase 6, mark its Requirements Summary entry with `from reference: <id>`. Step 4 of the procedure reads the template, copies it, and fills its slots via `Edit` calls.

## Out of scope

This skill writes **rule** templates only — never agent definitions (`.claude/agents/*.md`). Agent authoring is director's responsibility (see `agents/director.md`); when planning surfaces the need for a new agent, director writes it as part of plan execution.
