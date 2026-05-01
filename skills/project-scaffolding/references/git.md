# Git

## Required: Conventional Commits

This project uses [Conventional Commits](https://www.conventionalcommits.org/): `<type>(<scope>)?: <subject>`.

A structured-prefix commit convention is a hard requirement when the project uses the `plan-management` skill — director's `plan:` and `chore(ai):` commits, and the `Task:` body block, all assume one. Conventional Commits is the standard choice. If the project explicitly opted out during Discovery, an alternative structured-prefix convention must be used in its place; the reserved prefixes below still apply.

## Reserved prefixes (do not redefine)

These prefixes are owned by `plan-management` and must not be reused for unrelated purposes:

- `plan:` — director-only; empty commits that define or revise the plan.
- `chore(ai):` — director-only; empty commits that record task journal events (failures, supersessions, no-ops).
- `Task:` block in the commit body — director-only; per-task journal entries (`Outcome: passed | failed | superseded`).

Subagents dispatched by director never commit; director records all task outcomes after verification.

## Universal rules

- Keep subject lines ≤72 characters, imperative mood, lowercase after the colon.
- Explain the **why** in the body when the change isn't self-evident; the diff already shows the "what".
- Never `--amend` or force-push without explicit user approval. If a pre-commit hook fails, fix the cause and create a new commit — don't `--amend` over the failed one.
- Never `git push --force` to a shared branch (especially `main`).
- Never delete tags (`git tag -d`) or remote tags without explicit approval.
- Stage files by name (`git add <path>`), not `git add -A` or `git add .` — keeps stray `.env` files and credentials from slipping in.

## Project conventions

<!-- scaffolder fills these from Phase 5 / Phase 0 answers; remove any line that doesn't apply -->

- **Types in use**: <e.g. feat, fix, chore, docs, refactor, test, build, ci>
- **Scopes**: <e.g. api, ui, db — or "none required">
- **Release tags**: <e.g. `v<semver>`, or "this project does not tag releases">
- **Who commits**: <e.g. "director owns all commits" or "individual developers commit per branch policy">

## Examples

Good:
```
feat(api): add /users/me endpoint

Returns the authenticated user's profile so the client doesn't need
to look up its own id.
```

Bad — vague subject, past tense, capitalized after colon, body-less:
```
update stuff
feat: Added some improvements
fix: bug
```

<!-- scaffolder may add 1–2 project-specific examples below from git log observations -->
