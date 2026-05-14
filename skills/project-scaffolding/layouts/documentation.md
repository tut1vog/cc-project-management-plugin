# Documentation layout

Use when the project will accumulate research or rely on external reference material — true for most director-managed projects. Other layouts (e.g. `multi-agent`) build on top of this one.

## File tree

```
CLAUDE.md
.claude/
  rules/
    documentation.md
docs/
  research/
    README.md       ← index of research docs; director maintains it
  reference/        ← empty at scaffold time; human drops external files here
```

`docs/research/` holds agent-generated research output. `docs/reference/` holds human-provided external context (specs, vendor docs, exported material) — scaffolded as a bare directory; the user adds an index if they want one.

## `.claude/rules/documentation.md`

Create from `examples/documentation.md`. It auto-loads every session (no `paths:` frontmatter) so director always knows the `docs/` tree's ownership and where to find research and reference material.

## `docs/research/README.md` template

```markdown
# Research

Index of research documents in this directory. director appends an entry here whenever a research doc is saved.

- <!-- YYYY-MM-DD-topic.md — one-line summary -->
```

## CLAUDE.md

No `## Project Documentation` section is needed — the auto-loading `documentation.md` rule is director's channel for the `docs/` layout. Omit the section from `CLAUDE.md`.
