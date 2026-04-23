# Plan

## Status
Current phase: Phase 2: Wire agents to skillex (v0.2.0)
Current task: 2.1 — Wire cc-project-initializer to consult skillex during Discovery

---

## Phases

### Phase 1: Ship v0.1.0 ✅ complete
> Validate the uncommitted scaffold, document the plugin install workflow, commit the scaffold in logical Conventional Commits, and tag v0.1.0.

| ID | Task | Status |
|----|------|--------|
| 1.1 | Smoke-test the plugin end-to-end | done |
| 1.2 | Rewrite README.md for /plugin install workflow | done |

Phase shipped on 2026-04-23: annotated tag `v0.1.0` pushed to `origin` at commit `9b44bf8`. `main` tracks `origin/main`. Scaffold grouped into four Conventional Commits (feat(plugin), docs, chore, chore(ai)) plus the two passed-task commits for 1.1 and 1.2.


### Phase 2: Wire agents to skillex (v0.2.0)
> Update each agent's system prompt to call the skillex MCP tools (`mcp__skillex__search_skills`, `mcp__skillex__get_skill`, `mcp__skillex__list_repositories`). Initializer and advisor consult skillex during Discovery to surface relevant skills as suggestions; director consults it before dispatching an implementation task to check whether a matching skill already exists.

| ID | Task | Status |
|----|------|--------|
| 2.1 | Wire cc-project-initializer to consult skillex during Discovery | in-progress |
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

**ID**: 2.1
**Title**: Wire cc-project-initializer to consult skillex during Discovery
**Phase**: Phase 2: Wire agents to skillex (v0.2.0)
**Status**: in-progress

### Goal
Update `agents/cc-project-initializer.md` so the agent consults the bundled skillex MCP server during Discovery to surface relevant, already-written skills as candidate Claude Code features, rather than always proposing bespoke rule-file scaffolding. This is the reason skillex was bundled with the plugin.

### Context
- The agent file is at `agents/cc-project-initializer.md` (full body already read by the director before dispatch).
- Current structure: 3 modes (Discovery → Scaffold → Handoff). Discovery has 7 phases. Phase 6 is "Claude Code setup" — this is where the agent proposes `.claude/rules/*.md`, commands, agents, MCP, hooks, and the existing feature table already has a row for `.claude/skills/` (reusable prompt templates).
- The plugin ships a `skillex` MCP server (see `.mcp.json`) with tools: `mcp__skillex__search_skills`, `mcp__skillex__get_skill`, `mcp__skillex__list_repositories`. The default `SKILLS_MCP_REPOS` is `anthropics/skills`; users can override to replace the list (see `.claude/rules/mcp-config.md`).
- Project-agnostic body rule (`.claude/rules/agent-authoring.md`): the system prompt must not hardcode repo paths or project-specific assumptions. The skillex wiring must keep the agent usable in any consumer project.
- Over-calling concern noted in `project-brief.md` "Known Unknowns": "Over-calling adds latency and token cost; under-calling defeats the purpose of bundling it." The wiring should be judicious — a small number of targeted `search_skills` calls near Phase 6, plus at most one `get_skill` per promising hit when the user wants to learn more.
- The existing feature table row for `.claude/skills/` says: "Reusable prompt templates invoked as `/skill-name`; shareable across the team — When to suggest: Team has recurring prompts run on a regular cadence." This row must be updated to reflect that skillex now searches an external skill catalog, not just the project-local `.claude/skills/` directory.

### Implementation Steps
1. **Tools line** (line 12): add the three skillex MCP tool names to the declared tool set so the agent prompt is explicit about what it may call (per `agent-authoring.md`: "State the tools the agent may use explicitly inside the body"). Target string like: `Read, Write, Edit, Glob, Grep, Bash, WebSearch, WebFetch, mcp__skillex__search_skills, mcp__skillex__get_skill, mcp__skillex__list_repositories`.
2. **Feature table** (around line 25, the `.claude/skills/` row): update the "What it is" and "When to suggest" cells to mention that the plugin bundles skillex and that the agent will consult it during Phase 6 to surface relevant skills. Keep the row short.
3. **Discovery Phase 6** (around line 47, "Claude Code setup"): add a concrete instruction that before finalising the Claude Code setup proposal, the agent must call `mcp__skillex__search_skills` with 2–4 targeted queries derived from the Discovery answers (e.g. the primary language, the testing approach, the deployment target) and present any high-quality matches as candidate skills. The agent should optionally follow up with `mcp__skillex__get_skill` on a promising hit to show the user what the skill actually contains, but must not dump raw skill content into the summary — one-line headline + skillex id is enough. Note that skillex has no append semantics for `SKILLS_MCP_REPOS`; searches only cover whatever the user has configured.
4. **Requirements Summary template** (around line 82, the `Claude Code setup` bullet list): add a new sub-bullet for "Skills (from skillex)" that lists each matched skill id the user accepted, so the accepted skills flow into the handoff artifact.
5. **Scaffold Mode** (Mode 2, around Step 3 "Write `.claude/rules/*.md`"): the agent does not need to write skill files itself — skills come from the catalog. No scaffold change is required for skills beyond noting accepted skills in the summary. Do not introduce code that writes to a `.claude/skills/` directory.
6. Keep every edit surgical. Do not reshape the agent's 3-mode / 7-phase structure. Do not remove any existing content unless it directly contradicts the new skillex behaviour (e.g. the old `.claude/skills/` row's "When to suggest" text).

### Verification
- [ ] `grep -n "mcp__skillex__search_skills" agents/cc-project-initializer.md` returns at least one line (the Tools declaration) and a reference in the Discovery Phase 6 body.
- [ ] `grep -n "mcp__skillex__get_skill" agents/cc-project-initializer.md` returns at least one line.
- [ ] `grep -n "mcp__skillex__list_repositories" agents/cc-project-initializer.md` returns at least one line.
- [ ] The `.claude/skills/` row of the feature table still exists and has been updated to reference skillex.
- [ ] Discovery Phase 6 instructs the agent to call `search_skills` with queries derived from the user's Discovery answers before finalising the Claude Code setup proposal.
- [ ] The Requirements Summary template includes a "Skills" sub-bullet populated from skillex matches.
- [ ] The 3-mode structure (Discovery → Scaffold → Handoff) is intact; no existing phases were removed or renumbered.
- [ ] The body remains project-agnostic — no hardcoded repo paths or project-specific assumptions were introduced.
- [ ] The file still starts with valid YAML frontmatter: first line `---`, `name: cc-project-initializer`, non-empty `description:`.

### Suggested Agent
`general-purpose` — a focused system-prompt edit with clear targets. No specialist persona needed; the subagent just needs to follow the surgical plan above.
