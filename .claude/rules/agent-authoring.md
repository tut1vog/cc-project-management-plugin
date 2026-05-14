---
paths:
  - agents/**
---

# Agent authoring

- Each agent lives at `agents/<name>.md` with YAML frontmatter and a system prompt body.
- **Required frontmatter**: `name`, `description`, `tools:` (comma-separated; the harness enforces this list).
- One agent, one responsibility. Narrow scope + precise `description` makes auto-selection reliable.
- Cross-agent references use bare names (e.g. "hand off to director"), not plugin-namespaced paths.
- Declare preloaded skills via the `skills:` frontmatter field. Reserve on-demand reads for situationally relevant skills.
- Preload a skill only when it is *both* high-frequency across the agent's loop *and* cheap (small body). Everything else stays on-demand: name the skill at its point of use in the body so the agent reads it exactly when needed. A skill executed by a dispatched subagent is never preloaded into the dispatching agent — subagents don't inherit the `skills:` field.
- When editing an existing agent, preserve its contracts with other agents. `CLAUDE.local.md` (written by project-scaffolding) is auto-loaded at session start and consumed by director. `director` may maintain `PLAN.md` at the repo root; format in `skills/plan-management/SKILL.md`.

## Examples

Good:
```yaml
---
name: director
description: Autonomous technical director for complex multi-step work. Acts as a powerful actor — reads files, runs commands, edits code, and delegates to subagents only when warranted. Operates a continuous sense-act loop until the goal is met. Owns all git commits.
tools: Read, Write, Edit, Glob, Grep, Bash, WebSearch, WebFetch, Agent
---
```

Bad:
```yaml
---
name: Director          # wrong — must be kebab-case, must match filename
description: Helps.     # wrong — vague, no trigger signal for auto-selection
---
```
