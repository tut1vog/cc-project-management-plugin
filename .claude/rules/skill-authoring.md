---
paths:
  - skills/**
---

# Skill authoring

- Each skill is a directory at `skills/<name>/` containing a `SKILL.md` file.
- **Required frontmatter**: `name` (kebab-case, matches directory name) and `description`.
- A skill is a reusable prompt; when its format or commands change, only the skill file changes. When extracting content from an agent into a skill, remove the duplicated copy and replace with a pointer.
- Reference skills by bare name (e.g. "use the `plan-management` skill").
- Set `user-invocable: false` for background-knowledge skills preloaded by a specific agent via that agent's `skills:` field.
