# Agent authoring

**When to read this**: read before creating or editing any file under `agents/`.

## Rules
- Each agent is one file at `agents/<agent-name>.md` with YAML frontmatter plus a system prompt body.
- **Required frontmatter**: `name` (kebab-case, matches the filename stem) and `description` (one sentence, action-oriented — Claude Code auto-selects agents by matching this against the current task).
- The system prompt body must be **project-agnostic**. These agents are shipped in a plugin and dropped into unknown repos; hardcoding paths, tools, or project-specific assumptions breaks them for consumers.
- One agent, one responsibility. If an agent has two distinct jobs, split it into two agents. Narrow scope + precise `description` is what makes auto-selection reliable.
- State the tools the agent may use explicitly inside the body (e.g. "You have access to Read, Write, Edit, Glob, Grep, Bash, WebSearch, WebFetch."). Don't rely on implicit tool sets.
- Cross-agent references use bare agent names (e.g. "hand off to director"), not plugin-namespaced paths — subagents keep their bare names even when the plugin is installed.
- When editing an existing agent, preserve its contract with the others. `advisor` and `initializer` both produce `project-brief.md` for `director` to consume; changing that handoff format requires updating all three.
- After any edit to `agents/*.md`, verify the plugin still loads: `claude --plugin-dir ./` then `/agents` — all expected agents must appear.
- `initializer` and `advisor` deliberately share their Scaffold and Handoff sections, and all three agents share the skillex-consultation pattern. When editing one of these shared blocks, update every agent that carries it in the same commit. Shared blocks: Claude Code Features table, Phase 7 Director Permissions table, Requirements Summary fields, Scaffold Steps 1–5, Handoff template, and the skillex consultation paragraph.

## Examples

Good frontmatter:
```yaml
---
name: director
description: Autonomous project director for delegated end-to-end execution. Given a high-level goal, decomposes the work into a plan, dispatches subagents, verifies output, owns all git commits, and auto-continues through Orient → Plan → Dispatch → Verify with minimal user intervention.
---
```

Bad frontmatter:
```yaml
---
name: Director          # wrong — must be kebab-case, must match filename
description: Helps.     # wrong — vague, no trigger signal for auto-selection
---
```

Project-agnostic body (good):
> "Read `project-brief.md` at the working directory root to orient on the current task."

Project-specific body (bad — breaks for consumers):
> "Read `/home/ubun/github.com/tut1vog/cc-project-management-plugin/project-brief.md` to orient."
