# Plan

## Status
Current phase: Phase 1: Ship v0.1.0
Current task: 1.1 — Smoke-test the plugin end-to-end

---

## Phases

### Phase 1: Ship v0.1.0
> Validate the uncommitted scaffold, document the plugin install workflow, commit the scaffold in logical Conventional Commits, and tag v0.1.0.

| ID | Task | Status |
|----|------|--------|
| 1.1 | Smoke-test the plugin end-to-end | in-progress |
| 1.2 | Rewrite README.md for /plugin install workflow | pending |

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

## Current Task

**ID**: 1.1
**Title**: Smoke-test the plugin end-to-end
**Phase**: Phase 1: Ship v0.1.0
**Status**: in-progress

### Goal
Verify the uncommitted plugin scaffold is well-formed and loadable before we commit it or rewrite the README against it. This gates both the README rewrite (so the README can describe a verified install path) and the v0.1.0 tag.

### Context
- Project root: `/home/ubun/github.com/tut1vog/cc-project-management-plugin`.
- Scaffold is uncommitted — see `git status`. Everything below is on disk but not yet in `git log`.
- Files under test:
  - `.claude-plugin/plugin.json` — manifest; `version` must be `"0.1.0"`.
  - `.claude-plugin/marketplace.json` — single-plugin marketplace catalog; marketplace name `tut1vog-plugins`.
  - `.mcp.json` — must match the exact shape documented in `.claude/rules/mcp-config.md` (command `npx`, args `["-y", "github:tut1vog/skillex-mcp"]`, env `SKILLS_MCP_REPOS=anthropics/skills`).
  - `agents/cc-project-initializer.md`, `agents/cc-project-advisor.md`, `agents/cc-project-director.md` — each with YAML frontmatter where `name` is kebab-case matching the filename stem and `description` is a non-empty action-oriented sentence (per `.claude/rules/agent-authoring.md`).
- `npx -y github:tut1vog/skillex-mcp` depends on skillex-mcp's `prepare` script to compile TS at install time. First invocation on this machine may take 30–180s; subsequent runs are npx-cached. The brief confirms `prepare` is now landed on `skillex-mcp@main`.
- Consumer runtime requirement: Node ≥ 20.
- The fully authoritative end-to-end check — `claude --plugin-dir ./` followed by visual `/agents` and `/mcp` inspection — is **manual** per the brief ("No CI, no automated tests: validation is manual"). A subagent cannot drive an interactive Claude Code session, so the task must stop short of that step and hand a checklist back to the maintainer.

### Implementation Steps
1. `python -m json.tool .claude-plugin/plugin.json` — expect exit 0. Capture and report the `version` field.
2. `python -m json.tool .claude-plugin/marketplace.json` — expect exit 0.
3. `python -m json.tool .mcp.json` — expect exit 0. Confirm the structure matches `.claude/rules/mcp-config.md` exactly: one server named `skillex`, `command: "npx"`, `args` contains `"-y"` and `"github:tut1vog/skillex-mcp"`, `env.SKILLS_MCP_REPOS == "anthropics/skills"`.
4. For each of `agents/cc-project-initializer.md`, `agents/cc-project-advisor.md`, `agents/cc-project-director.md`: confirm the file starts with `---`, the frontmatter `name` matches the filename stem in kebab-case, and `description` is present and non-empty.
5. Smoke-test the skillex binary: `timeout 180 npx -y github:tut1vog/skillex-mcp --help 2>&1 | head -80`. A clean help output is ideal. If `--help` isn't supported, a "listening on stdio" / server-ready banner printed within a few seconds with no error trace also passes. A stack trace, non-zero exit, or 180s timeout fails this step.
6. Produce a short readiness report: per-check PASS/FAIL with the observed output snippet, plus a manual-validation checklist for the maintainer — `claude --plugin-dir ./` → `/agents` shows all three agents → `/mcp` shows `skillex` running.

### Verification
- [ ] `python -m json.tool` exits 0 on all three JSON files.
- [ ] `plugin.json` `version` field equals `"0.1.0"`.
- [ ] `.mcp.json` matches the shape in `.claude/rules/mcp-config.md` exactly.
- [ ] All three agent files present with valid frontmatter (`name:` kebab-case matching filename stem; non-empty `description:`).
- [ ] `npx -y github:tut1vog/skillex-mcp --help` (or a clean startup banner) runs without error within 180s.
- [ ] Readiness report produced with per-check PASS/FAIL and the manual-validation checklist for the maintainer.

### Suggested Agent
`general-purpose` — straightforward file/JSON validation plus one `npx` invocation. No specialist knowledge required; no code is written.
