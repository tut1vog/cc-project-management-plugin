# Multi-agent layout

Use when the goal decomposes into clearly separate, autonomous responsibilities — each handled by a dedicated agent with its own workflows and memory.

## File tree

```
CLAUDE.md
.claude/
  rules/
  agents/
    <agent-a>.md
    <agent-b>.md
docs/
  <agent-a>/
    README.md
    workflows/      ← empty at scaffold time
    memory/         ← empty at scaffold time
  <agent-b>/
    README.md
    workflows/
    memory/
```

`CLAUDE.md` and `.claude/rules/` serve as shared knowledge for all agents. Per-agent documentation lives under `docs/<agent>/`.

## `.claude/agents/<agent>.md` template

```markdown
---
name: <agent-name>
description: <one sentence — action-oriented, used for auto-selection>
tools: <comma-separated list>
---

For domain knowledge, workflows, and accumulated memory, read `docs/<agent-name>/README.md`.

<agent-specific knowledge and constraints>
```

## `docs/<agent>/README.md` template

```markdown
# <Agent Name>

<One-line description of this agent's responsibility.>

## Workflows

- <!-- link or describe workflows as they are added -->

## Memory

- <!-- link or describe memory entries as they are added -->
```

## CLAUDE.md addition

Add the following line to `CLAUDE.md` so any agent can locate per-agent documentation:

```markdown
> Per-agent documentation (workflows and memory) lives under `docs/<agent>/`.
```
