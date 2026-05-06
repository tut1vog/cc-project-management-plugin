---
paths:
  - agents/**
  - skills/**
---

# Agent and skill authoring

## Rules (agents)
- Each agent is one file at `agents/<agent-name>.md` with YAML frontmatter plus a system prompt body.
- **Required frontmatter**: `name` (kebab-case, matches the filename stem) and `description` (one sentence, action-oriented — Claude Code auto-selects agents by matching this against the current task).
- The system prompt body must be **project-agnostic**. These agents are shipped in a plugin and dropped into unknown repos; hardcoding paths, tools, or project-specific assumptions breaks them for consumers.
- One agent, one responsibility. If an agent has two distinct jobs, split it into two agents. Narrow scope + precise `description` is what makes auto-selection reliable.
- Declare the agent's tools in the `tools:` frontmatter field (comma-separated). The harness enforces this list; the body should reference tools only when describing usage, not as a sidebar list. Director uses bare `Agent` to dispatch to any agent the consumer project ships.
- Cross-agent references use bare agent names (e.g. "hand off to director"), not plugin-namespaced paths — subagents keep their bare names even when the plugin is installed.
- When editing an existing agent, preserve its contracts with the others. `CLAUDE.local.md` (written by project-scaffolding skill execution) is auto-loaded by Claude Code at session start and consumed by director; gitignored via `*.local.*`. `director` does not write to `CLAUDE.local.md` — it reads and maintains `PLAN.md` at the repo root, with the format documented in `skills/plan-management/SKILL.md`.
- When an agent always uses a skill, declare it via the `skills:` frontmatter field. The harness preloads the skill content into the agent's context at startup, so the agent does not need to be told to read it. Reserve on-demand reads for skills that are only situationally relevant.
- After any edit to `agents/*.md`, verify the plugin still loads: `claude --plugin-dir ./` then `/agents` — all expected agents must appear.

## Rules (skills)
- Each skill is a directory at `skills/<skill-name>/` containing a `SKILL.md` file. The directory name and the frontmatter `name` must match (kebab-case).
- **Required frontmatter**: `name` (kebab-case) and `description` (one or two sentences, action-oriented, including the situations and trigger phrases that should auto-select the skill — Claude Code auto-selects skills the same way it auto-selects agents).
- The skill body must be **project-agnostic**. Skills ship in the plugin and run inside unknown consumer repos; never hard-code paths or assume project-specific tools.
- A skill is a reusable prompt that any agent can read; if the format or commands a skill describes change, only the skill file should change. When extracting content into a skill, remove the duplicated copy from the agent body and replace with a pointer.
- When an agent depends on a skill, reference it by bare name (e.g. "use the `plan-management` skill") — same convention as cross-agent references.
- A skill that exists only as background knowledge for a specific agent (preloaded via that agent's `skills:` field) sets `user-invocable: false` so it does not appear in the user's `/` menu. Skills a user might reasonably invoke directly leave the field at its default.
- After any edit to `skills/<name>/SKILL.md`, verify the plugin still loads with `claude --plugin-dir ./` and the skill is discoverable.

## Examples

Good frontmatter:
```yaml
---
name: director
description: Autonomous project director for delegated end-to-end execution. Given a high-level goal, decomposes the work into a plan, dispatches subagents, verifies output, owns all git commits, and auto-continues through Orient → Plan → Dispatch → Verify with minimal user intervention.
tools: Read, Write, Edit, Glob, Grep, Bash, WebSearch, WebFetch, Agent
skills:
  - plan-management
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
> "Read `CLAUDE.local.md` at the working directory root to orient on the current task."

Project-specific body (bad — breaks for consumers):
> "Read `/home/ubun/github.com/tut1vog/cc-project-management-plugin/CLAUDE.local.md` to orient."
