---
name: lint-instructions
description: Lints instruction and documentation files for five smells — Duplication, Legacy Negation, Verbosity, Ambiguous Subjectivity, and Decorative Directive — and applies all fixes in place immediately without reporting. Use when auditing or cleaning up CLAUDE.md, AGENTS.md, SKILL.md, prompt files, or similar instruction docs.
---

# lint-instructions

1. Identify the target: a file, glob, or directory of instruction/documentation files. If nothing was named, ask once.
2. Read every targeted file in full. For directories, enumerate first.
3. Evaluate every smell type in every targeted file without exception.
4. For Duplication, run a cross-file pass comparing every targeted file against every other.
5. Apply all fixes in place immediately without a findings report or approval prompt.

## The five smells

1. **Duplication** — the same rule stated in two or more places (paraphrases count); keep one authoritative location and replace the rest with a one-line reference.
2. **Legacy Negation** — a directive anchored to a prior state ("no longer", "anymore", "previously"); rewrite as the current desired behavior and delete the historical contrast.
3. **Verbosity** — background explanation or implementation detail that belongs in a code comment or appendix, not operational instructions; reduce to the actionable directive.
4. **Ambiguous Subjectivity** — vague, unmeasurable adjectives ("good", "robust", "clean", "appropriate") whose compliance cannot be checked by reading the output; replace with a measurable constraint.
5. **Decorative Directive** — a line that suppresses no named failure mode and whose removal produces no observable change; delete it, or if deletion would regress behavior, rewrite it to explicitly forbid that failure mode.

## Scope guards

- Skip binary files, lockfiles, and any path matching `*.local.*` or `.env*`.
- Skip `PLAN.md` and `MEMORY.md` — these are data files, not instruction files.
- Do not lint this skill's own `SKILL.md` unless the user explicitly targets it.
- If the target is a directory, restrict to common instruction file types by default: `*.md`, `AGENTS`, `CLAUDE.md`, `SKILL.md`, prompt files. Ask before extending to other file types.
