# Git

**When to read this**: read before making any commit or tag.

## Rules
- Use [Conventional Commits](https://www.conventionalcommits.org/): `<type>(<scope>)?: <subject>`. Types in use here: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `build`, `ci`, plus director-only `plan`. The `chore(ai):` scope is reserved for director's task-journal bookkeeping commits (failures, supersessions, no-op events). The `plan:` type is reserved for director's empty plan-definition commits — its body is the full phases/tasks decomposition; see `skills/plan-management/SKILL.md` for the canonical body schema and read/write commands.
- Keep subject lines ≤72 chars, imperative mood, lowercase after the colon.
- Explain the **why** in the body when the change isn't self-evident. The "what" is the diff.
- Never `--amend` or force-push without explicit user approval. If a pre-commit hook fails, fix the cause and create a new commit — don't `--amend` over it.
- Never `git push --force` to `main`. Never delete tags (`git tag -d`) or remote tags without explicit approval.
- Stage files by name (`git add <path>`), not `git add -A` or `git add .`, so a stray `.env` or credential can't slip in.
- Release tags follow `v<semver>` (e.g. `v0.1.0`, `v0.2.0`). The tag must be cut on a commit where `.claude-plugin/plugin.json`'s `version` matches. See `releasing.md`.
- director owns all git commits on this repo; other subagents dispatched by it do not commit.

## Examples

Good commits:
```
feat(mcp): wire skillex server with anthropics/skills as default index

Ships .mcp.json so the plugin boots with a working skill catalog
out of the box. Users override via SKILLS_MCP_REPOS env var.
```
```
docs: rewrite README install section for /plugin install workflow
```
```
chore(ai): record task 3.2 — agent-authoring rule verified
```
```
plan: initialise plan for replacing plan.md with plan: commits

Goal: move director's plan from a tracked file into git history.

## Phase 1: Director rewrite
> Update director.md to read/write plans via `plan:` commits.
- 1.1 rewrite director.md plan-storage section
- 1.2 update advisor/initializer cascading references

## Phase 2: Verify end-to-end
> Confirm plugin loads and director can plan against the new model.
- 2.1 run claude --plugin-dir ./ and inspect /agents
```

Bad commits:
```
update stuff                   # vague, no type
feat: Added some improvements  # past tense, capitalized, vague
fix: bug                       # body-less, uninformative
```
