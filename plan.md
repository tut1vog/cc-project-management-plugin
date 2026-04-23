# Plan

## Status
Current phase: Phase 1: Ship v0.1.0
Current task: 1.2 — Rewrite README.md for /plugin install workflow

---

## Phases

### Phase 1: Ship v0.1.0
> Validate the uncommitted scaffold, document the plugin install workflow, commit the scaffold in logical Conventional Commits, and tag v0.1.0.

| ID | Task | Status |
|----|------|--------|
| 1.1 | Smoke-test the plugin end-to-end | done |
| 1.2 | Rewrite README.md for /plugin install workflow | in-progress |

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

**ID**: 1.2
**Title**: Rewrite README.md for /plugin install workflow
**Phase**: Phase 1: Ship v0.1.0
**Status**: in-progress

### Goal
Replace the legacy copy-paste install instructions in `README.md` with the `/plugin install` workflow so consumers can install the plugin from the marketplace, and document the prerequisites and skills configuration the plugin requires at runtime. This is the last content change before the v0.1.0 tag.

### Context
- Project root: `/home/ubun/github.com/tut1vog/cc-project-management-plugin`.
- Current `README.md` (6459 bytes) still describes the pre-plugin copy-paste workflow. Preserve its existing agent descriptions and the "How They Work Together" narrative — only the install story and runtime-prerequisites story should change.
- Marketplace identifiers (shipped in `.claude-plugin/marketplace.json`): marketplace name is `tut1vog-plugins`; the plugin is `cc-project-management-plugin`.
- The install flow consumers will use:
  ```
  /plugin marketplace add github:tut1vog/cc-project-management-plugin
  /plugin install cc-project-management-plugin@tut1vog-plugins
  ```
- Prerequisites to document:
  - Claude Code with plugin support.
  - **Node ≥ 20** on the end-user machine (required by `npx` to launch `skillex-mcp`). First launch of the plugin takes 30–120s as skillex-mcp's `prepare` script compiles TypeScript at install time; subsequent launches are npx-cached.
- Skills configuration to document (per `.claude/rules/mcp-config.md`):
  - The plugin ships `.mcp.json` with `SKILLS_MCP_REPOS=anthropics/skills` as the baked-in default.
  - `SKILLS_MCP_REPOS` is **comma-separated** and whatever a user sets **replaces** the default — skillex has no append semantics. To add more repos, users set the env var to a list that still includes `anthropics/skills` if they want to keep it.
  - Example:
    ```bash
    # replace the default entirely
    SKILLS_MCP_REPOS="anthropics/skills,myorg/my-skills" claude
    # or set persistently in a shell profile
    export SKILLS_MCP_REPOS="anthropics/skills,myorg/my-skills"
    ```
- What must survive the rewrite:
  - The agent descriptions for cc-project-initializer, cc-project-advisor, cc-project-director.
  - The "How They Work Together" pipeline explanation.
- What must be **removed or rewritten**:
  - Any copy-paste-`.md`-files installation instructions.
  - Anything implying the agents are delivered as loose files rather than a plugin.

### Implementation Steps
1. Read the current `README.md` end-to-end to map its existing sections.
2. Rewrite the install/usage story:
   - Replace the legacy "Installation" section with the `/plugin marketplace add` → `/plugin install` flow above.
   - Add a "Prerequisites" section listing Claude Code + Node ≥ 20 + a note on first-run skillex build time.
   - Add a "Configuring skills" section explaining `SKILLS_MCP_REPOS` replacement semantics (NOT append) and showing both the one-off and the persistent override forms.
3. Preserve the existing agent descriptions and the "How They Work Together" section verbatim, or with only light editing for consistency with the new install story.
4. Write the result back to `README.md`. Do not touch any other file.

### Verification
- [ ] `README.md` contains the exact marketplace-add command for `github:tut1vog/cc-project-management-plugin`.
- [ ] `README.md` contains the exact install command `/plugin install cc-project-management-plugin@tut1vog-plugins`.
- [ ] `README.md` has a "Prerequisites" section that explicitly says **Node ≥ 20** and mentions the first-run skillex build time.
- [ ] `README.md` has a "Configuring skills" section that states `SKILLS_MCP_REPOS` **replaces** the default (does not extend it) and shows a concrete override example.
- [ ] `README.md` no longer contains any copy-paste-`.md`-files install instructions (grep for phrases like "copy", ".md file" in an install context — should be absent from install sections).
- [ ] The three agent descriptions and the "How They Work Together" narrative still present and readable.

### Suggested Agent
`general-purpose` — straightforward Markdown rewrite with a specific brief; no specialist persona needed.
