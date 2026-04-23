# Claude Subagents Library

A collection of reusable Claude subagent definitions for use across projects.

## Repository Structure

```
.claude/
  agents/
    <agent-name>.md   # One file per agent
CLAUDE.md             # Project instructions (this file)
README.md             # Usage guide for consumers
```

## Agent File Format

Each agent is defined in `.claude/agents/<agent-name>.md`:

```markdown
---
name: <agent-name>
description: <one-line description — used by Claude to decide when to invoke this agent>
---

<system prompt — role, capabilities, constraints, behavior>
```

### Sections

| Section | Required | Purpose |
|---|---|---|
| YAML frontmatter (`name`, `description`) | Yes | Claude uses `description` to auto-select the agent |
| System prompt body | Yes | Agent role, tools allowed, and behavior — kept project-agnostic |

### Writing good agents

- **Project-agnostic body**: The system prompt must contain no project-specific details.
- **Focused scope**: One agent, one responsibility. Prefer narrow, composable agents.
- **Precise description**: Claude reads `description` to decide whether to invoke the agent — make it action-oriented and specific.
- **Explicit tools**: State which tools the agent may use.

