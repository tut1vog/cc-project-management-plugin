---
paths:
  - agents/**
  - skills/**
---

# Authoring

- Names are kebab-case and must match the filename stem (agents) or directory name (skills).
- `description` must be action-oriented with trigger phrases — Claude Code auto-selects agents and skills by matching this field against the current task.
- Bodies must be project-agnostic. These artifacts ship in a plugin and run inside unknown consumer repos; never hard-code paths or assume project-specific tools.
- After any edit, verify the plugin still loads: `claude --plugin-dir ./`.
- Describe the current state only. Never mention removed, renamed, or superseded artifacts — even as negations. Exception: runtime context the reader will actively encounter (e.g. a prior failed task during a retry).

## Example

Project-agnostic (good):
> "Read `CLAUDE.local.md` at the working directory root to orient on the current task."

Project-specific (bad — breaks for consumers):
> "Read `/home/ubun/github.com/tut1vog/cc-project-management-plugin/CLAUDE.local.md` to orient."
