---
name: plan-management
description: Read and write the project plan that director maintains in PLAN.md at the repo root. Use when you need to know the current plan, derive task status, find the next pending task, or update task status after a task event. Triggers on "current plan", "task status", "what's been done", "is this task in progress", "mark task done".
---

The project plan lives in **`PLAN.md`** at the repo root. Director creates and maintains this file using the Read and Edit tools. No helper command or script is required.

## PLAN.md format

```
Goal: <high-level goal description>

## Phase 1: <Phase Name>

- [ ] T1: <task title>
- [x] T2: <task title>
- [!] T3: <task title>

## Phase 2: <Phase Name>

- [ ] T4: <task title>
```

- `Goal:` is the first line of the file, followed by a blank line.
- Each phase is a `##` heading, followed by a blank line and a task list.
- Task IDs (`T1`, `T2`, …) are stable and assigned monotonically. Never reuse a removed ID.
- File order is execution order — no dependency pointer syntax.

## Task status markers

| Marker | Meaning |
|---|---|
| `- [ ] Tn: <title>` | not started |
| `- [x] Tn: <title>` | done |
| `- [!] Tn: <title>` | failed |

## Reading the plan

Read PLAN.md with the Read tool. To find the next task to dispatch, scan for the first `- [ ]` line.

## Writing the plan

Director edits PLAN.md with the Edit tool. Every PLAN.md edit is part of a regular Conventional Commit — no separate plan-only commits.

| Event | Edit |
|---|---|
| Task passed verification | Change `- [ ]` to `- [x]` |
| Task failed verification | Change `- [ ]` to `- [!]` |
| Remediation task added | Insert `- [ ] T<next>: <title>` after the failed task |
| New goal planned | Create or overwrite PLAN.md with the full plan |
| Plan revised (new tasks, phases) | Add/remove/reorder lines directly |
| Direction change removes tasks | Delete the affected task lines |

## Assigning stable IDs

Scan PLAN.md for the highest existing `T<n>` number and increment by one for each new task. Phases have no ID prefix — their order in the file is their order.

## Example

```
Goal: Add user-facing auth using short-lived JWTs with refresh tokens.

## Phase 1: Token issuance

- [x] T1: add /auth/login endpoint with credential validation
- [x] T2: issue JWT + refresh token on successful login
- [!] T3: add tests for /auth/login
- [ ] T6: add tests for /auth/login (remediation — scope mocks correctly)

## Phase 2: Token validation middleware

- [ ] T4: add express-jwt middleware to /api routes
- [ ] T5: add tests for middleware
```
