---
name: plan-management
description: Read, parse, and (for director only) write the project plan and per-task outcomes that the director stores in git history. Use when you need to know the current plan, derive task status, find a prior task's outcome, or compose a `plan:` or `Task:` journal commit body. Triggers on "current plan", "task status", "what's been done", "plan commit", "task journal", "is this task in progress".
---

The project plan and task journal live entirely in **git commit history**. Every agent in a director-managed project reads plan state by querying `git log` and the `plan-management` helper. This skill is the canonical spec for that format and the read/write commands; agents and humans should treat it as the single source of truth.

## Who writes / who reads

- **Director writes.** Director is the only agent that lands `plan:` and `Task:` commits. Subagents dispatched by director never commit; they report back and director records the outcome.
- **Anyone reads.** Any agent — director, a dispatched subagent, an ad-hoc agent in the same project — can run the read commands below to inspect the current plan or task history. Reading needs `Bash`, `git`, and Python 3 for the `plan-management` helper described in **Reading the current plan**.

## Storage model

- The **current plan** is anchored at the most recent commit whose subject begins with `plan:` (the chain leaf). It is reconstructed by walking `Parent:` references back to a root (a `plan:` commit with no `Parent:`) and applying each commit's deltas in chain order. A new root supersedes every prior `plan:` commit.
- **Task outcomes** (passed / failed / superseded) live in the body of every commit director makes after a task event — including code-bearing commits (`feat:`, `fix:`, etc.) and empty bookkeeping commits (`task:`).
- Both `plan:` and `task:` commits are typically `--allow-empty` — they carry no file changes, only the journal in the body.

## Identity model

- Phases carry **stable opaque IDs** (`P1`, `P2`, `P3`, …) assigned monotonically at creation. IDs never change once assigned and are never recycled, even after a task or phase is removed.
- Tasks carry **stable opaque IDs** (`T1`, `T2`, `T3`, …) global across the entire plan (not phase-local), assigned monotonically at creation.
- **Display labels** (`Phase 1`, `Phase 2`; tasks `1.2`, `1.3`, …) are derived by the helper at read time from the resolved phase order plus task order within phase. Display labels can change between revisions; stable IDs cannot. Always reference stable IDs in commit bodies and dispatch prompts; render display labels only when communicating with the user.

## `plan:` commit format

**Subject** (≤72 chars): `plan: <summary>` — e.g. `plan: initialise plan for shopping-cart MVP`, `plan: revise after task T7 failure`.

A `plan:` commit body is either a **root** (declares the full plan, no `Parent:`) or a **child** that chains off a previous `plan:` commit by SHA.

### Root body

```
Goal: <high-level goal description>

## Phase P1: <Phase Name>
> <One-sentence goal for this phase>
+ T1: <task title>
+ T2: <task title>

## Phase P2: <Phase Name>
> <One-sentence goal for this phase>
+ T3: <task title>
```

Rules for roots:

- `Goal:` is required and authoritative for the whole chain.
- Every phase block carries `## Phase Pn: <title>` followed by an optional `> <goal>` line and a list of `+ Tn: <title>` add-task ops.
- A `+ Tn` op with no `[after:]` clause appends to the phase's task list. To position explicitly, write `+ Tn: <title> [after: Tm]`.
- A `## Phase Pn` header with no `[after:]` clause appends to the plan's phase list. To position explicitly, write `## Phase Pn: <title> [after: Pm]`.
- Plan bodies are **lean**: task titles only, no per-task implementation detail. Rich per-task context (files to touch, verification steps, warnings) is composed fresh in the dispatch prompt at execution time and is never persisted to the plan body.

### Child body

A child carries only the **deltas** against its parent — adds, revises, removes, and reorders. Phases not mentioned in the child are inherited unchanged.

```
Parent: <full-40-char-sha-of-previous-plan-commit>

## Phase P3: <new phase title> [after: P1]
> <one-sentence goal>
+ T9: <task title>
+ T10: <task title>

## Phase P1
+ T11: <new task in existing phase> [after: T2]
~ T3: <revised title>
- T5

- P2

Notes: <optional — why this revision>
```

Rules for children:

- `Parent:` MUST be the first non-empty line of the body. Its presence is the sole signal that the commit is chained.
- Children omit `Goal:` (inherited from the root).
- A `## Phase Pn` header **resolves against the chain**: if `Pn` already exists, it's a delta on that phase; if not, it's a new phase declaration. Same syntax for both — the helper distinguishes by chain context, no mode marker.
- Inside a phase block, the operators are:

  | Op | Meaning |
  |---|---|
  | `+ Tn: <title>` | add task `Tn`; appends to the phase by default |
  | `+ Tn: <title> [after: Tm]` | add task `Tn`, positioned after `Tm` |
  | `~ Tn: <new title>` | revise task `Tn`'s title |
  | `~ Tn [after: Tm]` | reorder task `Tn` without changing its title |
  | `~ Tn: <new title> [after: Tm]` | revise title and reorder in one op |
  | `- Tn` | remove task `Tn` from the plan |

- Phase header forms in a child:

  | Form | Meaning |
  |---|---|
  | `## Phase Pn: <title> [after: Pm]` | declare new `Pn` (full block: `> goal` + `+` tasks follow) **or** revise existing `Pn`'s title and/or order |
  | `## Phase Pn: <title>` | declare new `Pn` or revise existing `Pn`'s title |
  | `## Phase Pn [after: Pm]` | reorder existing `Pn` without changing its title |
  | `## Phase Pn` | no header change; lines below operate on `Pn`'s tasks only |
  | `> <goal>` (under `## Phase Pn`) | declare or revise `Pn`'s goal |

- Phase removal at the **top level** of a child body:

  | Op | Meaning |
  |---|---|
  | `- Pn` | remove phase `Pn` and all its tasks from the plan |

  Phase removal ops appear at the top level of the commit body (not inside a phase block). When `- Pn` removes a phase, every phase that was `[after: Pn]` is automatically promoted to point at `Pn`'s `after` target. All tasks inside `Pn` are also removed, with their `after` pointers promoted in the same way.

- A phase declaration in a child must use only `+` task ops; a delta on an existing phase may mix `+`, `~`, `-`.
- `Notes:` is optional in children. When present it captures why this revision exists and is shown by `plan-management notes` as part of the chain's revision history. `Notes:` is per-revision and is not merged into the body.
- The body is **parse-strict**: any line that is not blank, `Parent:`, `Goal:`, `Notes:` (and its continuation), `## Phase`, `>` goal, `- Pn` (phase removal), or one of the task operators is a parse error. This is the structural enforcement of the lean-bodies rule — prose belongs in `Notes:` only.

### Ordering rules

- A new phase or task with **no `[after:]`** clause appends after the most recently-created sibling at the time the op is applied.
- An explicit `[after: X]` places the item directly after `X`.
- **Tiebreak**: when several items share the same `[after:]` target, the earlier-created one displays first (queue semantics — items appear in the order they were added).
- The helper rejects bodies with cycles in `after:` pointers, dangling references (`after: T99` when `T99` doesn't exist), and cross-phase task pointers (`+ T_new [after: <task in different phase>]`).

### Removing a task that other tasks point to

When `- Tn` removes a task, every other task that was `[after: Tn]` is automatically promoted to point at `Tn`'s `after:` target. Removing a task is therefore safe — the chain of remaining tasks stays intact without the user needing to fix up sibling pointers.

### When to chain vs start a new root

- **Chain** for any additive or local change — appending a phase, inserting one mid-plan, adding/revising/removing tasks, removing a phase, reordering.
- **Start a new root** when the goal itself changes, or when the plan needs structural reorganization the delta vocabulary doesn't express cleanly.

## `Task:` journal commit body

Every commit director creates after a task event carries this block in the body, below the subject and a blank line:

```
Task: T7 (1.2) — <title>
Outcome: passed | failed | superseded
What was done: <1–3 sentences>
Verification: <checks ran and their result; "n/a" for supersessions/bookkeeping>
Notes: <non-obvious context: skipped steps, edge cases>
```

`T7` is the task's stable ID — that's what readers query against. `(1.2)` is the display label at the time of writing — convenience for skimming `git log`. The label may become stale if the plan reorders later; the stable ID never lies. For remediation subtasks added via `+ T_new` after a failure, the new task gets the next monotonic stable ID and a fresh display label derived from its position in the phase.

## Reading the current plan

The chain-walk and merge are encapsulated in the `plan-management` command, shipped on the Bash tool's `PATH` whenever this plugin is enabled. Run it from inside the project's git repository.

```bash
# Print the merged plan body with derived display labels and stable IDs annotated.
plan-management current

# Print the merged plan annotated with per-task status (done / failed / superseded).
plan-management status

# Print the chain root SHA (use it as the lower bound for task-status queries).
plan-management root

# Print the chain leaf SHA (the most recent plan: commit).
plan-management leaf

# Print every chain commit SHA, one per line, root → leaf.
plan-management chain

# Given a stable ID (Pn or Tn), print its current display label and title.
plan-management resolve T7

# Print the per-revision Notes blocks from chain commits, with their commit SHAs and subjects.
plan-management notes

# Print the next available stable IDs as "P<n> T<m>" — use to pick fresh IDs when
# composing a new plan: commit body.
plan-management next-ids

# Parse-and-cycle-check the current chain. Exits 0 (prints "ok") on success;
# non-zero with a diagnostic on parse errors, dangling references, or cycles.
plan-management validate
```

The command exits with code 2 and a stderr message if no `plan:` commit exists in history, code 3 on parse errors or chain inconsistencies. Code 1 means invalid arguments. A `plan:` commit without `Parent:` is a length-1 chain — root and leaf are the same SHA, and `current` prints that root's resolved body.

The output of `plan-management current` annotates each phase header with `[Pn]` and each task line with `[Tn]` so the reader sees both the human-friendly display label (`1.2`) and the stable ID (`T7`) inline.

The output of `plan-management resolve` uses the format `Phase N: <title>` for phases and `Task N.M: <title>` for tasks.

## Deriving task status

Use `plan-management status` for a fully-joined view. To query manually, walk every task event since the chain root using the task's stable ID:

```bash
# Enumerate every Task: journal entry since the chain root
ROOT=$(plan-management root)
git log "$ROOT"..HEAD --grep="^Task:" --format="%H%n%B%n----"

# Every event for one task by stable ID (across all plans)
git log --grep="^Task: T7"
```

Then classify each task ID from the merged plan:

- **done** — a `Task: Tn` entry with `Outcome: passed` exists in the window.
- **failed** — the most recent task event for `Tn` has `Outcome: failed` and no later `passed` entry exists.
- **superseded** — the most recent task event for `Tn` has `Outcome: superseded`.
- Tasks with no journal entry in the window have not been started.

Task entries before the current chain's root belong to a superseded plan and should be ignored when computing status against the current plan.

## Querying task history

```bash
# Every event for a specific task by stable ID
git log --grep="^Task: T7"

# Show the body of a specific task event
git show <sha>

# Every plan: commit (chronological history of plan revisions)
git log --grep="^plan:"

# Every task event since a given plan
git log <plan-sha>..HEAD --grep="^Task:"

# All commits except director's bookkeeping (filter out task: noise)
git log --invert-grep --grep="^task:"
```

## Writing commits (director only)

Director uses three commit shapes. The shape depends on the task outcome and whether the plan itself is changing.

| Shape | When | Subject | Body | Empty? |
|---|---|---|---|---|
| **Passed task** | Subagent code passed verification | Project-convention subject (`feat:`, `fix:`, `docs:` …) | `Task: Tn (<label>) — <title>` block with `Outcome: passed` | No — bundles subagent code + director's doc updates |
| **Failed task or single-task supersession** | Verification failed, or a task is superseded by a direction change | `task: T4 failed — <title>` or `task: T4 superseded — <reason>` | `Task: Tn (<label>) — <title>` block with `Outcome: failed` or `Outcome: superseded` | Yes — `git commit --allow-empty` |
| **Plan creation or revision** | New plan, or revision triggered by a failure / direction change | `plan: <summary>` | Either a root body (Goal / Phases / Notes) or a chained child body (`Parent: <sha>` + delta ops) per the formats above | Yes — `git commit --allow-empty` |

**Picking fresh stable IDs**: run `plan-management next-ids` and use the returned values for any new phase or task. Never reuse a removed ID.

**Ordering when a failure triggers plan revision**: land the `task:` failure journal first, then a separate `plan:` commit reflecting the revised plan.

**Superseding the original failed task after remediation**: when a remediation task (added via `+ T<next>` after a failure) passes verification, write a `task: T<failed> superseded — remediated by T<next>` empty commit for the original failed task immediately before auto-continuing. This closes the loop: the original task moves from `failed` to `superseded` in the status history.

**Reserved prefixes**: `plan:` is reserved for plan-definition commits. `task:` is reserved for director's task-journal bookkeeping (failures, supersessions, no-file-change events). Both are documented in the project's commit-conventions rule (typically `.claude/rules/git.md`); read that rule before composing any director commit so the subject line follows the project's overall convention (Conventional Commits or otherwise).

**Subagents do not commit.** If you are dispatched into a task, do not run `git commit` yourself — director records the outcome after verification.

## Examples

### A root `plan:` commit body

```
plan: initialise plan for JWT authentication

Goal: Add user-facing auth using short-lived JWTs with refresh tokens.

## Phase P1: Token issuance
> Implement and test the /auth/login flow end-to-end.
+ T1: add /auth/login endpoint with credential validation
+ T2: issue JWT + refresh token on successful login
+ T3: add tests for /auth/login (happy path + invalid credentials)

## Phase P2: Token validation middleware
> Protect existing /api routes with JWT validation.
+ T4: add express-jwt middleware to /api routes
+ T5: add tests for middleware (valid / expired / malformed tokens)
```

### A chained `plan:` commit body — mid-plan phase insert + per-task delta

```
plan: insert refresh-rotation phase + remediation subtask after T4 failure

Parent: a1b2c3d4e5f60718293a4b5c6d7e8f9012345678

## Phase P3: Refresh token rotation [after: P1]
> Rotate refresh tokens on every use to mitigate token theft.
+ T6: add rotation logic to refresh endpoint
+ T7: add tests for refresh-rotation

## Phase P2
+ T8: split /api router so /auth/* sits outside the jwt middleware [after: T4]

Notes: P3 inserted between P1 and P2 after the rotation requirement landed.
T8 added to P2 to remediate task T4 — the previous attempt scoped the
middleware too broadly and locked out /auth/login itself.
```

After this commit, `plan-management current` produces:

```
Goal: Add user-facing auth using short-lived JWTs with refresh tokens.

## Phase 1: Token issuance [P1]
> Implement and test the /auth/login flow end-to-end.
- 1.1 [T1] add /auth/login endpoint with credential validation
- 1.2 [T2] issue JWT + refresh token on successful login
- 1.3 [T3] add tests for /auth/login (happy path + invalid credentials)

## Phase 2: Refresh token rotation [P3]
> Rotate refresh tokens on every use to mitigate token theft.
- 2.1 [T6] add rotation logic to refresh endpoint
- 2.2 [T7] add tests for refresh-rotation

## Phase 3: Token validation middleware [P2]
> Protect existing /api routes with JWT validation.
- 3.1 [T4] add express-jwt middleware to /api routes
- 3.2 [T8] split /api router so /auth/* sits outside the jwt middleware
- 3.3 [T5] add tests for middleware (valid / expired / malformed tokens)
```

### A passed-task commit body

```
feat(auth): add JWT issuance on /auth/login

Task: T2 (1.2) — issue JWT + refresh token on successful login
Outcome: passed
What was done: Added jwt.sign call returning a 15-minute access token and a
30-day refresh token. Refresh token persisted to the sessions table.
Verification: `curl -X POST /auth/login -d '{...}'` returns 200 with both
tokens; refresh row appears in the sessions table.
Notes: Refresh-token rotation is intentionally deferred to Phase 2 (P3).
```

### A failed-task `task:` commit body

```
task: T4 failed — add express-jwt middleware to /api routes

Task: T4 (3.1) — add express-jwt middleware to /api routes
Outcome: failed
What was done: Wired express-jwt at the app level; all /api routes now
require a token, including /api/auth/login itself.
Verification: `curl -X POST /api/auth/login` returns 401 — login endpoint
unreachable to unauthenticated callers, breaking the entire flow.
Notes: Need to scope middleware to a sub-router that excludes /auth/*.
Remediation will land as a new task added to P2 (display label TBD after
the next plan: commit assigns it).
```

### A supersession `task:` commit body

```
task: T4 superseded — remediated by T8

Task: T4 (3.1) — add express-jwt middleware to /api routes
Outcome: superseded
What was done: n/a
Verification: n/a
Notes: T8 completed the remediation; T4 is closed.
```
