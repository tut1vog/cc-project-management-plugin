---
name: plan-management
description: Read, parse, and (for director only) write the project plan and per-task outcomes that the director stores in git history. Use when you need to know the current plan, derive task status, find a prior task's outcome, or compose a `plan:` or `Task:` journal commit body. Triggers on "current plan", "task status", "what's been done", "plan commit", "task journal", "is this task in progress".
---

The project plan and task journal live entirely in **git commit history**. Every agent in a director-managed project reads plan state by querying `git log`. This skill is the canonical spec for that format and the read/write commands; agents and humans should treat it as the single source of truth.

## Who writes / who reads

- **Director writes.** Director is the only agent that lands `plan:` and `Task:` commits. Subagents dispatched by director never commit; they report back and director records the outcome.
- **Anyone reads.** Any agent — director, a dispatched subagent, an ad-hoc agent in the same project — can run the read commands below to inspect the current plan or task history. Reading needs `Bash`, `git`, and Python 3 for the `plan-management` helper described in **Reading the current plan**.

## Storage model

- The **current plan** is anchored at the most recent commit whose subject begins with `plan:` (the chain leaf). It is either that commit's body (a root) or the body merged from the chain it sits on, walked back through `Parent: <sha>` references to a root. A new root (a `plan:` commit with no `Parent:`) supersedes every prior `plan:` commit.
- **Task outcomes** (passed / failed / superseded) live in the body of every commit director makes after a task event — including code-bearing commits (`feat:`, `fix:`, etc.) and empty bookkeeping commits (`chore(ai):`).
- Both `plan:` and `chore(ai):` commits are typically `--allow-empty` — they carry no file changes, only the journal in the body.

## `plan:` commit format

**Subject** (≤72 chars): `plan: <summary>` — e.g. `plan: initialise plan for shopping-cart MVP`, `plan: revise after task 2.1 failure`.

**Body**:

```
Goal: <high-level goal description>

## Phase 1: <Phase Name>
> <One-sentence goal for this phase>
- 1.1 <task title>
- 1.2 <task title>

## Phase 2: <Phase Name>
> <One-sentence goal for this phase>
- 2.1 <task title>

Notes: <optional — supersession reasons, decisions made, why the plan changed>
```

Plan bodies are intentionally **lean**: task titles only, no per-task implementation detail. Rich per-task context (files to touch, verification steps, warnings) is composed fresh in the dispatch prompt at execution time and is **never** persisted to the plan body or any other file.

### Root vs chained `plan:` commits

A `plan:` commit is either a **root** (full body, exactly the schema above) or a **child** that chains off a previous `plan:` commit by SHA. A child carries only the new or revised phase blocks; everything else is inherited from the chain.

**Child body**:

```
Parent: <full-40-char-sha-of-previous-plan-commit>

## Phase N: <new or revised phase>
> <One-sentence goal>
- N.x <task title>

Notes: <required — why this revision>
```

Rules:

- `Parent:` MUST be the first non-empty line of the body. Its presence is the sole signal that the commit is chained.
- Children omit `Goal:`. The root's goal is authoritative for the whole chain. If the goal itself changes, write a new root (no `Parent:`) and re-state the full plan.
- A child's `## Phase N:` block with the same N as a phase in the chain **overrides** that phase; a new N **appends**. Phases not mentioned in the child are inherited unchanged.
- `Notes:` is per-revision and is not merged — readers display each commit's notes alongside the merged plan as a revision history.
- Phase deletion is not supported in a child. To remove phases, write a new root.

**When to chain vs start a new root**:

- Chain for additive or local changes — appending a phase, revising the tasks of one phase, recording a debugging phase after a failure.
- Start a new root when the high-level goal changes, when phases must be deleted, or when the plan needs structural reorganization.

## `Task:` journal commit body

Every commit director creates after a task event carries this block in the body, below the subject and a blank line:

```
Task: N.M — <title>
Outcome: passed | failed | superseded
What was done: <1–3 sentences>
Verification: <checks ran and their result; "n/a" for supersessions/bookkeeping>
Notes: <non-obvious context: skipped steps, edge cases>
```

`N.M` is the task ID from the current plan body (e.g. `1.2`). For remediation subtasks added after a failure, use a third number (e.g. `1.2.1`).

## Reading the current plan

The chain-walk and merge are encapsulated in the `plan-management` command, shipped on the Bash tool's `PATH` whenever this plugin is enabled. Run it from inside the project's git repository.

```bash
# Print the merged plan body (Goal + phase blocks sorted by N)
plan-management current

# Print the chain root SHA (use it as the lower bound for task-status queries)
plan-management root

# Print the chain leaf SHA (the most recent plan: commit)
plan-management leaf

# Print every chain commit SHA, one per line, root → leaf
plan-management chain
```

The command exits with code 2 and a stderr message if no `plan:` commit exists in history. A `plan:` commit without `Parent:` is a length-1 chain — root and leaf are the same SHA, and `current` prints that commit's body verbatim.

Merge semantics applied by `current`: walk root → leaf, take `Goal:` from the root, keep a phase dictionary keyed by phase number where a later commit's `## Phase N:` block overrides any earlier block for the same N, emit `Goal:` followed by phase blocks sorted by N ascending. `Notes:` lines are per-revision and are not merged into the body — read individual chain commits with `git show <sha>` when a revision history is needed.

## Deriving task status

For each task ID listed in the **merged** plan, walk every task event since the chain root:

```bash
# Enumerate every Task: journal entry since the chain root
ROOT=$(plan-management root)
git log "$ROOT"..HEAD --grep="^Task:" --format="%H%n%B%n----"
```

Then classify each task ID from the merged plan:

- **done** — a `Task: N.M` entry with `Outcome: passed` exists in the window.
- **in-progress** — the most recent task event for `N.M` is `Outcome: failed` or `Outcome: superseded` and no later passed entry exists. The task was attempted but has not yet been redispatched and passed.
- **pending** — no task event for `N.M` exists in the window.

Task entries before the current chain's root belong to a superseded plan and should be ignored when computing status against the current plan, but remain accessible via the queries below for historical lookups.

## Querying task history

```bash
# Every event for a specific task ID (across all plans)
git log --grep="Task: N.M"

# Show the body of a specific task event
git show <sha>

# Every plan: commit (chronological history of plan revisions)
git log --grep="^plan:"

# Every task event since a given plan
git log <plan-sha>..HEAD --grep="^Task:"

# All commits except director's bookkeeping (filter out chore(ai): noise)
git log --invert-grep --grep="chore(ai)"
```

## Writing commits (director only)

Director uses three commit shapes. The shape depends on the task outcome and whether the plan itself is changing.

| Shape | When | Subject | Body | Empty? |
|---|---|---|---|---|
| **Passed task** | Subagent code passed verification | Project-convention subject (`feat:`, `fix:`, `docs:` …) | `Task: N.M` block with `Outcome: passed` | No — bundles subagent code + director's doc updates |
| **Failed task or single-task supersession** | Verification failed, or a task is superseded by a direction change | `chore(ai): record task N.M failure — <title>` or `chore(ai): supersede task N.M — <reason>` | `Task: N.M` block with `Outcome: failed` or `Outcome: superseded` | Yes — `git commit --allow-empty` |
| **Plan creation or revision** | New plan, or revision triggered by a failure / direction change | `plan: <summary>` | Either a root body (Goal / Phases / Notes) or a chained child body (`Parent: <sha>` + only new/revised phases) per the format above | Yes — `git commit --allow-empty` |

**Ordering when a failure triggers plan revision**: land the `chore(ai):` failure journal first, then a separate `plan:` commit reflecting the revised plan.

**Reserved prefixes**: `plan:` is reserved for plan-definition commits. `chore(ai):` is reserved for director's task-journal bookkeeping. Both are documented in the project's commit-conventions rule (typically `.claude/rules/git.md`); read that rule before composing any director commit so the subject line follows the project's overall convention (Conventional Commits or otherwise).

**Subagents do not commit.** If you are dispatched into a task, do not run `git commit` yourself — director records the outcome after verification.

## Examples

### A root `plan:` commit body

```
plan: initialise plan for JWT authentication

Goal: Add user-facing auth using short-lived JWTs with refresh tokens.

## Phase 1: Token issuance
> Implement and test the /auth/login flow end-to-end.
- 1.1 add /auth/login endpoint with credential validation
- 1.2 issue JWT + refresh token on successful login
- 1.3 add tests for /auth/login (happy path + invalid credentials)

## Phase 2: Token validation middleware
> Protect existing /api routes with JWT validation.
- 2.1 add express-jwt middleware to /api routes
- 2.2 add tests for middleware (valid / expired / malformed tokens)
```

### A chained `plan:` commit body

```
plan: extend with debugging phase after task 2.1 repeated failure

Parent: a1b2c3d4e5f60718293a4b5c6d7e8f9012345678

## Phase 3: Middleware scoping debug
> Diagnose why express-jwt rejects /auth/* and land a scoped fix.
- 3.1 reproduce 401 on /auth/login with a minimal repro server
- 3.2 split /api router so /auth/* sits outside the jwt middleware
- 3.3 regression test that /auth/login is reachable unauthenticated

Notes: Added after task 2.1 failed twice (see chore(ai) commits 9f8e7d6,
4c5b6a7). Phases 1–2 carry over from the parent unchanged.
```

### A passed-task commit body

```
feat(auth): add JWT issuance on /auth/login

Task: 1.2 — issue JWT + refresh token on successful login
Outcome: passed
What was done: Added jwt.sign call returning a 15-minute access token and a
30-day refresh token. Refresh token persisted to the sessions table.
Verification: `curl -X POST /auth/login -d '{...}'` returns 200 with both
tokens; refresh row appears in the sessions table.
Notes: Refresh-token rotation is intentionally deferred to Phase 3.
```

### A failed-task `chore(ai):` commit body

```
chore(ai): record task 2.1 failure — express-jwt middleware

Task: 2.1 — add express-jwt middleware to /api routes
Outcome: failed
What was done: Wired express-jwt at the app level; all /api routes now
require a token, including /api/auth/login itself.
Verification: `curl -X POST /api/auth/login` returns 401 — login endpoint
unreachable to unauthenticated callers, breaking the entire flow.
Notes: Need to scope middleware to a sub-router that excludes /auth/*.
Remediation will land as task 2.1.1.
```
