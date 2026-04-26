# No legacy references

**When to read this**: read before editing any prose in `agents/`, `skills/`, `.claude/rules/`, `docs/`, `README.md`, or `CLAUDE.md`.

## Rules
- Describe the current state only. Do not mention removed, renamed, or superseded artifacts (files, flags, conventions, behaviors) — even to negate them.
- Negation still anchors the reader to the obsolete model and adds noise. Drop the clause; the positive statement stands on its own.
- Do not introduce a "formerly", "previously", "no longer", "instead of", "used to be", "deprecated", or "migrated from" gloss when removing an obsolete reference. Just remove it.
- Exception: a reference is **not** legacy if it describes runtime context that the reader will actively encounter — e.g. director's handling of a *prior failed task event* is current behavior, not history. The test is "will the reader meet this thing while doing the work?"
- Commit messages and the plan/task journal are exempt — those are explicitly historical records and should describe what changed and why.
- Use the rule retroactively: when editing prose for any other reason, take the chance to strip any legacy clauses you notice in the surrounding lines.

## Examples

Bad — negation that anchors the reader to the obsolete artifact:
> The plan lives in git history — no `plan.md`, no `tasks.json`.
> The dispatch prompt is composed fresh; we no longer read from `state.json`.
> Use the new `plan-management` skill instead of the old hardcoded format.

Good — current state only:
> The plan lives in git history.
> The dispatch prompt is composed fresh at execution time.
> Use the `plan-management` skill for plan format and read commands.

Allowed — runtime context, not legacy:
> If this task is a retry or remediation of a previously failed task, …
(The previously-failed task is a real event the reader will look up in `git log` to compose the dispatch prompt — current workflow, not history.)
