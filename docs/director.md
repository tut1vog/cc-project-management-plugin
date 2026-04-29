# director

A senior technical project director agent that plans, tracks, and verifies implementation work across sessions. It never writes application code — it coordinates subagents that do.

## What it does

- Reads the latest `plan:` commit in git history, recent task journal entries, and current task context on every invocation (Orient)
- Decomposes goals into phases and tasks, one agent session per task (Plan)
- Produces self-contained prompts for the subagent that will do the work (Dispatch)
- Runs verification checks after each task and lands the outcome in a commit body (Verify)
- Updates project documentation after verification passes, so docs never drift from the code — reading `.claude/rules/documentation.md` (or whichever rule file handles docs in the project's Rules index) on demand
- Reads behavioral rules from `.claude/rules/*.md` on demand, using the Rules index in `CLAUDE.md` to decide which to load for the current task
- Stores the live plan in git history as empty `plan:` commits and records every task outcome in a `Task:` journal entry in the corresponding commit body — `git log` is the single source of truth

## Usage

Invoke the director by name in Claude Code. It always runs Orient silently first — using the bundled `plan-management` skill to read the latest `plan:` commit and derive task status from `Task:` journal entries, then reading the files implicated by any in-progress task. `CLAUDE.md` is loaded automatically by Claude Code at session start, and `CLAUDE.local.md` is loaded automatically when present, so the director already has durable project facts and the current goal without an explicit read step.

### Start a new plan

```
Use director to plan: add user authentication with JWT
```

The director will:
1. Orient (read the latest `plan:` commit, derive task status from journal entries)
2. Ask clarifying questions only if needed
3. Propose phases and tasks
4. Present the plan and wait for your confirmation
5. On confirmation, land an empty `plan:` commit with the full body and dispatch the first task to a subagent

The director auto-continues through Orient → Plan → Dispatch → Verify without stopping, except whenever the plan itself needs to change — a new plan, a remediation after failed verification, an infeasible task, or a course correction from new information. In each of those cases it surfaces the revised plan and waits for user review before resuming. The execution framework automatically runs the dispatched subagent, returns the results, and the director verifies — looping through dispatch → execute → verify until the plan is complete.

### Verify and advance

After each subagent completes, the execution framework returns the results to the director. It runs the verification steps it defined when dispatching, then either:
- **Passes**: updates any affected project documentation (reading the documentation rule file from `.claude/rules/` when one is indexed in `CLAUDE.md`), and creates a single commit bundling the subagent's code and the doc updates — subject per project conventions, passed journal entry in the body. Auto-continues to the next pending task in the plan.
- **Fails**: lands a `chore(ai):` empty commit with the failure journal in the body. If the failure requires a plan revision (remediation subtask, split, restructure), proposes the revised plan; on confirmation lands a new `plan:` empty commit before dispatching the remediation. Stops and waits for user review.
- **Infeasible**: discards partial changes (`git checkout -- <paths>`), lands a `chore(ai):` empty commit recording the failure, proposes a restructured plan, and on confirmation lands a new `plan:` commit. Stops and waits for user validation.

### Change direction mid-plan

```
Use director — we're dropping OAuth, going with magic links instead
```

The director identifies which pending tasks are superseded, presents the impact, lands `chore(ai):` empty commits with `Outcome: superseded` for each affected task, then lands a new `plan:` empty commit reflecting the revised plan.

## Persistent memory

| Location | Purpose | Edited by |
|----------|---------|-----------|
| Latest `plan:` commit body | Current plan — phases and task titles | Director (each `plan:` commit supersedes the previous) |
| `Task:` journal entries (commit bodies) | Per-task outcomes — passed, failed, superseded | Director (via commits; commits are immutable) |

The canonical format spec, read commands (latest plan, task status, task history), and write templates all live in the bundled **`plan-management`** skill. Any agent in a director-managed project can read the plan and task journal by following that skill — it requires only `Bash`, no plugin-specific tools. The illustrative examples below are kept here for skim-readers; if they ever drift from the skill, the skill is authoritative.

### `plan:` commit body — example

Plan bodies are intentionally lean (task titles only); rich per-task context is composed fresh in the dispatch prompt at execution time and never persisted.

```
plan: <≤72-char summary>

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

### Task journal commit body — example

Every commit director creates — whether it carries code or is `--allow-empty` — has a journal entry in its body:

```
<subject line per project conventions>

Task: 1.2 — Add JWT middleware
Outcome: passed
What was done: Added express-jwt middleware to all /api routes. Token validation reads the secret from process.env.JWT_SECRET.
Verification: `curl -H "Authorization: Bearer invalid" /api/me` returns 401. `curl -H "Authorization: Bearer <valid>" /api/me` returns 200 with user payload.
Notes: Excluded /api/auth/login and /api/auth/refresh from the middleware chain.
```

## Git behaviour

**Subagents never make git commits — the director owns all git operations.** Git conventions come from whatever `.claude/rules/*.md` file the project's Rules index points at for commits (typically `.claude/rules/git.md`); the director reads it on demand before its first commit of the session. If no such rule is indexed, it falls back to conventional commits. Body templates for `plan:` and `Task:` journal entries are defined in the `plan-management` skill — the table below covers only the subject line per shape.

The director uses three commit shapes:

- **Passed task**: one commit bundling subagent code + director's doc updates. Subject per project conventions (`feat:`, `fix:`, `docs:`).
- **Failed task or single-task supersession**: `git commit --allow-empty` with a `chore(ai):` subject and the failure / supersession journal in the body.
- **Plan creation or revision**: `git commit --allow-empty` with a `plan:` subject and the full plan body. After a failure that triggers a plan revision, this lands as a second commit, after the `chore(ai):` failure journal.

| Event | Commit subject |
|-------|----------------|
| New plan created | `plan: initialise plan for <goal>` |
| Plan revised (intent change or failure) | `plan: revise after <reason>` |
| Task verified (pass) | Project-convention subject (e.g. `feat: <summary>`); code + docs in one commit |
| Task verified (fail) | `chore(ai): record task N.M failure — <task title>` (followed by a `plan:` commit if remediation requires plan revision) |
| Task superseded by direction change | `chore(ai): supersede task N.M — <reason>` (followed by a `plan:` commit with the revised plan) |

Director bookkeeping commits use the `chore(ai):` prefix so they can be filtered from git log (e.g. `git log --invert-grep --grep="chore(ai)"`). Plan commits are isolated with `git log --grep="^plan:"`.

## Workflow overview

```
You ──→ advisor ──→ CLAUDE.local.md
                                                              |
                                                              ▼
                                                        director
                                                              |
                                                          Orient
                                                          Plan (land plan: commit)
                                                              |
                                                     ┌──── Dispatch ────→ Subagent
                                                     │        |          [implements]
                                                     │        ▼
                                                     │     Verify (run checks)
                                                     │     commit (journal entry in body)
                                                     │        |
                                                     └────────┘  (loop until plan complete)
```

## Tips

- **The repo is the source of truth.** Since the plan lives in `plan:` commits and every task outcome is recorded in a commit body, any collaborator can resume where you left off by running `git log -n 1 --grep="^plan:"` and `git log --grep="^Task:"`.
- **One task at a time.** The director enforces this. Don't try to run two tasks in parallel — verification is sequential by design.
- **Verification is non-negotiable.** Every task must have a runnable check. If the director can't write one, the task needs more decomposition. When verification runs tests, the director doesn't trust a green pass alone — it reads the test source to confirm meaningful assertions, adequate coverage of code paths, and that mocks don't bypass real logic.
- **Separate authors for code and tests.** Implementation and test-writing are always separate tasks dispatched to different agents. The agent that wrote the code never writes its own tests — this ensures independent verification.
- **New agents are rare.** The director defaults to `general-purpose` and only proposes a new subagent when no existing agent covers a task's narrow, recurring responsibility. When one is needed, it's drafted in the plan proposal and written to `.claude/agents/` (YAML frontmatter + project-agnostic body) after you confirm — before the first task that depends on it is dispatched.
- **Docs travel with the code.** On every passing verification, the director assesses whether the change affects user-facing documentation and edits the relevant files itself before the code commit, reading the documentation rule file (e.g. `.claude/rules/documentation.md`) on demand when one is indexed in `CLAUDE.md`. Purely internal refactors skip this step; anything observable (new features, changed behavior, new commands, new configuration) gets documented in the same commit as the code.
- **Let the director re-dispatch on failure.** When verification fails, the director revises the plan and produces a corrected prompt. Don't skip this step and re-run the old prompt manually.
- **Changing direction is safe.** Just tell the director what changed. It will identify the impact, ask for confirmation, and supersede the affected tasks before doing anything else.
