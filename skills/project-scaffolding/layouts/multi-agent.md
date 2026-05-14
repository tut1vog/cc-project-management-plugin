# Multi-agent layout

Use when the goal decomposes into clearly separate, autonomous responsibilities — each handled by a dedicated agent with its own workflows and memory.

This layout **extends the `documentation` layout**. Apply `documentation` first (it creates `docs/research/`, `docs/reference/`, and `.claude/rules/documentation.md`), then add the per-agent structure below.

## File tree

```
.claude/
  agents/
    <agent-a>.md
    <agent-b>.md
docs/
  agents/
    <agent-a>/
      README.md
      workflows/      ← empty at scaffold time
      memory/         ← empty at scaffold time
    <agent-b>/
      README.md
      workflows/
      memory/
```

`CLAUDE.md` and `.claude/rules/` serve as shared knowledge for all agents. Per-agent documentation lives under `docs/agents/<agent>/`.

## `.claude/agents/<agent>.md` template

```markdown
---
name: <agent-name>
description: <one sentence — action-oriented, used for auto-selection>
tools: <comma-separated list>
---

For domain knowledge, workflows, and accumulated memory, read `docs/agents/<agent-name>/README.md`.

<agent-specific knowledge and constraints>
```

## `docs/agents/<agent>/README.md` template

```markdown
# <Agent Name>

<One-line description of this agent's responsibility.>

## Workflows

- <!-- link or describe workflows as they are added -->

## Memory

- <!-- link or describe memory entries as they are added -->
```

## `documentation.md` addition

Append a clause to `.claude/rules/documentation.md` so its ownership map covers the per-agent tree:

```markdown
- `docs/agents/<agent>/` — per-agent workflows and memory, managed by the owning agent.
```
