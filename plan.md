# Plan

## Status
Current phase: Phase 1: Ship v0.1.0 — closeout
Current task: Phase 1 closeout — tag v0.1.0 and push (awaiting user confirmation per `.claude/rules/releasing.md`)

---

## Phases

### Phase 1: Ship v0.1.0
> Validate the uncommitted scaffold, document the plugin install workflow, commit the scaffold in logical Conventional Commits, and tag v0.1.0.

| ID | Task | Status |
|----|------|--------|
| 1.1 | Smoke-test the plugin end-to-end | done |
| 1.2 | Rewrite README.md for /plugin install workflow | done |

**Scaffold committed up front (director-only, pre-dispatch):** per user direction on 2026-04-23 the scaffold is committed **before** task 1.1 runs rather than at phase close. Grouping per `project-brief.md` item 3 and `.claude/rules/git.md`:
- `feat(plugin):` plugin structure — `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json`, the three `agents/cc-project-*.md` renames from `.claude/agents/`.
- `docs:` documentation refresh — `CLAUDE.md`, `.claude/rules/*.md`, `project-brief.md` (force-added; gitignored by default).
- `chore:` metadata — `LICENSE`, `.gitignore`.
- `chore(ai):` `plan.md` initialisation (force-added; gitignored by default).

**Phase 1 closeout (director-only, after 1.1 and 1.2 pass — not dispatched):**
- `docs:` README rewrite commit bundled with task 1.2 verification per passed-task rule.
- Confirm `.claude-plugin/plugin.json` `version` is `0.1.0`.
- Tag `v0.1.0` and push per `.claude/rules/releasing.md` — both `git tag` and `git push` require explicit user confirmation on this public repo.

### Phase 2: Wire agents to skillex (v0.2.0)
> Update each agent's system prompt to call the skillex MCP tools (`search_skills`, `get_skill`, `list_repositories`). Initializer and advisor consult skillex during Discovery to surface relevant skills as suggestions; director consults it before dispatching an implementation task to check whether a matching skill already exists.

| ID | Task | Status |
|----|------|--------|
| 2.1 | Wire cc-project-initializer to consult skillex during Discovery | pending |
| 2.2 | Wire cc-project-advisor to consult skillex during Discovery | pending |
| 2.3 | Wire cc-project-director to consult skillex before dispatching implementation tasks | pending |

Each task is dispatched to a distinct subagent per the brief's directive ("Each agent edit is its own task, assigned to a separate subagent from whoever runs the subsequent verification"). Verification for each task is done by the director, not another subagent.

**Phase 2 closeout (director-only):**
- Bump `.claude-plugin/plugin.json` to `0.2.0`.
- Tag `v0.2.0` and push (user confirms).

### Phase 3: Post-release housekeeping (optional)
> Scheduled only if a specific need arises. Candidates from the brief: `CHANGELOG.md`, `CONTRIBUTING.md`, a GitHub Actions workflow that validates JSON on PRs. No tasks pre-allocated.

---

## Phase 1 Closeout (director-only, awaiting user confirmation)

Both dispatched tasks in Phase 1 are passed and committed. What remains is the release itself, which requires explicit user confirmation per `.claude/rules/releasing.md`:

1. Confirm `.claude-plugin/plugin.json` `version` is `0.1.0` at `HEAD` (already true — no bump needed).
2. **User-confirmed:** `git tag v0.1.0` on the head commit.
3. **User-confirmed:** `git push origin main --tags`.

Once the tag is pushed, Phase 1 is complete and the director advances to Phase 2 task 2.1 (wire cc-project-initializer to skillex).
