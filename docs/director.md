# director

A senior technical project director agent that plans, tracks, and verifies implementation work across sessions. It never writes application code — it coordinates subagents that do.

## What it does

- Reads `plan.md`, recent git history, and current task context on every invocation (Orient)
- Decomposes goals into phases and tasks, one agent session per task (Plan)
- Produces self-contained prompts for the subagent that will do the work (Dispatch)
- Runs verification checks after each task and updates the plan accordingly (Verify)
- Updates project documentation after verification passes, so docs never drift from the code — reading `.claude/rules/documentation.md` (or whichever rule file handles docs in the project's Rules index) on demand
- Reads behavioral rules from `.claude/rules/*.md` on demand, using the Rules index in `CLAUDE.md` to decide which to load for the current task
- Maintains `plan.md` as current state and records every task outcome in a git commit message body — `git log` is the long-term journal

## Installation

Copy `.claude/agents/director.md` from this repo into your project's `.claude/agents/` directory. No other setup is required.

```
your-project/
  .claude/
    agents/
      director.md   ← copy here
  plan.md                      ← created by the director (current state)
```

Long-term history lives in `git log` — every task outcome is recorded in a commit message body.

## Usage

Invoke the director by name in Claude Code. It always runs Orient silently first — reading `plan.md`, recent git history (`git log --oneline -20`, then `git show <sha>` for relevant commits), and any in-progress task's context files — before responding. `CLAUDE.md` is loaded automatically by Claude Code at session start, so the director has durable project facts (including the Rules index for on-demand rule loading) without needing an explicit read step.

### Start a new plan

```
Use director to plan: add user authentication with JWT
```

The director will:
1. Orient (read project state)
2. Ask clarifying questions only if needed
3. Write `plan.md`
4. Present the plan and wait for your confirmation
5. On confirmation, commit `plan.md` and dispatch the first task to a subagent

The director auto-continues through Orient → Plan → Dispatch → Verify without stopping, except whenever the plan itself needs to change — a new plan, a remediation after failed verification, an infeasible task, or a course correction from new information. In each of those cases it surfaces the revised plan and waits for user review before resuming. The execution framework automatically runs the dispatched subagent, returns the results, and the director verifies — looping through dispatch → execute → verify until the plan is complete.

### Verify and advance

After each subagent completes, the execution framework returns the results to the director. It runs the verification steps from `plan.md`, then either:
- **Passes**: marks the task done in `plan.md`, updates any affected project documentation (reading the documentation rule file from `.claude/rules/` when one is indexed in `CLAUDE.md`), and creates a single commit bundling the subagent's code, the doc updates, and the `plan.md` update — subject per project conventions, passed journal entry in the body. Auto-continues to the next task.
- **Fails**: diagnoses the gap, revises `plan.md` with a remediation subtask, commits the revision with a `chore(ai):` subject and failed journal entry in the body (using `git commit --allow-empty` if `plan.md` didn't actually change), then stops and waits for user review.
- **Infeasible**: if the subagent reported the task as impossible, discards partial changes, revises the plan, commits the revision (or an `--allow-empty` `chore(ai):` commit) with the failed journal entry, then stops and waits for user validation.

### Change direction mid-plan

```
Use director — we're dropping OAuth, going with magic links instead
```

The director identifies which pending tasks are superseded, presents the impact, then revises the plan and commits the revision with a `chore(ai):` subject and a supersession journal entry in the body.

## Persistent memory

| Location | Purpose | Edited by |
|----------|---------|-----------|
| `plan.md` | Current state — active phase, task detail, verification steps | Director (overwritten each update) |
| `git log` | Long-term journal — every task outcome recorded in a commit message body | Director (via commits; commits are immutable) |

`plan.md` is committed to git automatically. To review recent work, use `git log --oneline -20` and `git show <sha>`; to find a specific task's history, `git log --grep="Task: N.M"`.

### plan.md structure

```markdown
# Plan

## Status
Current phase: <phase name>
Current task: <task id> — <task title>

---

## Phases

### Phase 1: <Phase Name>
> <One-sentence goal>

| ID  | Task         | Status     |
|-----|--------------|------------|
| 1.1 | <task title> | done       |
| 1.2 | <task title> | in-progress|
| 1.3 | <task title> | pending    |

---

## Current Task

**ID**: 1.2
**Title**: <task title>
**Phase**: <phase name>
**Status**: in-progress

### Goal
<What this task accomplishes and why.>

### Context
<Relevant files, patterns, constraints — specific paths and names.>

### Implementation Steps
1. <Concrete step>
2. <Next step>

### Verification
- [ ] <Runnable command or observable output>

### Suggested Agent
<agent name> — <why this agent fits>
```

### Commit body format

Every commit the director creates — whether it carries code, a `plan.md` revision, or is `--allow-empty` — has a journal entry in its body:

```
<subject line per project conventions>

Task: 1.2 — Add JWT middleware
Outcome: passed
What was done: Added express-jwt middleware to all /api routes. Token validation reads the secret from process.env.JWT_SECRET.
Verification: `curl -H "Authorization: Bearer invalid" /api/me` returns 401. `curl -H "Authorization: Bearer <valid>" /api/me` returns 200 with user payload.
Notes: Excluded /api/auth/login and /api/auth/refresh from the middleware chain.
```

## Git behaviour

**Subagents never make git commits — the director owns all git operations.** Git conventions come from whatever `.claude/rules/*.md` file the project's Rules index points at for commits (typically `.claude/rules/git.md`); the director reads it on demand before its first commit of the session. If no such rule is indexed, it falls back to conventional commits.

Every commit carries the task's journal entry in its body. The director uses three commit shapes:

- **Passed task**: one commit bundling subagent code + director's doc updates + `plan.md` update. Subject per project conventions (`feat:`, `fix:`, `docs:`).
- **Failed task or supersession**: one commit of the `plan.md` revision with a `chore(ai):` subject.
- **No file changes** (e.g. discarded subagent work left `plan.md` untouched): `git commit --allow-empty` with a `chore(ai):` subject so the event still lands in `git log`.

Director bookkeeping commits use the `chore(ai):` prefix so they can be filtered from git log (e.g. `git log --invert-grep --grep="chore(ai)"`).

| Event | Commit subject |
|-------|----------------|
| New plan created | `chore(ai): initialise plan for <goal>` |
| Plan revised (intent change) | `chore(ai): revise plan — supersede tasks X, Y` |
| Task verified (pass) | Project-convention subject (e.g. `feat: <summary>`); code + docs + `plan.md` in one commit |
| Task verified (fail, plan revised) | `chore(ai): revise plan after failed verification of N.M — <task title>` |

## Workflow overview

```
You ──→ initializer / advisor ──→ project-brief.md
                                                              |
                                                              ▼
                                                        director
                                                              |
                                                          Orient
                                                          Plan (write plan.md, commit)
                                                              |
                                                     ┌──── Dispatch ────→ Subagent
                                                     │        |          [implements]
                                                     │        ▼
                                                     │     Verify (run checks)
                                                     │     update plan.md
                                                     │     commit (journal entry in body)
                                                     │        |
                                                     └────────┘  (loop until plan complete)
```

## Tips

- **Commit your plan file.** Since `plan.md` is auto-committed and every task outcome is recorded in a commit body, the repo itself is the source of truth — any collaborator can resume where you left off by reading `plan.md` and `git log`.
- **One task at a time.** The director enforces this. Don't try to run two tasks in parallel — verification is sequential by design.
- **Verification is non-negotiable.** Every task must have a runnable check. If the director can't write one, the task needs more decomposition. When verification runs tests, the director doesn't trust a green pass alone — it reads the test source to confirm meaningful assertions, adequate coverage of code paths, and that mocks don't bypass real logic.
- **Separate authors for code and tests.** Implementation and test-writing are always separate tasks dispatched to different agents. The agent that wrote the code never writes its own tests — this ensures independent verification.
- **New agents are rare.** The director defaults to `general-purpose` and only proposes a new subagent when no existing agent covers a task's narrow, recurring responsibility. When one is needed, it's drafted in the plan proposal and written to `.claude/agents/` (YAML frontmatter + project-agnostic body) after you confirm — before the first task that depends on it is dispatched.
- **Docs travel with the code.** On every passing verification, the director assesses whether the change affects user-facing documentation and edits the relevant files itself before the code commit, reading the documentation rule file (e.g. `.claude/rules/documentation.md`) on demand when one is indexed in `CLAUDE.md`. Purely internal refactors skip this step; anything observable (new features, changed behavior, new commands, new configuration) gets documented in the same commit as the code.
- **Let the director re-dispatch on failure.** When verification fails, the director revises the plan and produces a corrected prompt. Don't skip this step and re-run the old prompt manually.
- **Changing direction is safe.** Just tell the director what changed. It will identify the impact, ask for confirmation, and update the plan cleanly before doing anything else.
