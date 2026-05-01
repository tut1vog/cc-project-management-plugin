---
name: skill-catalog
description: Discover pre-built skills (SKILL.md files) in trusted GitHub repositories via the `gh` CLI. Use when proposing reusable skills during project setup or when looking for an existing skill that solves the current problem. Triggers on "find a skill", "search skills", "is there a skill for", "skill catalog".
---

A bundled skill that searches the user's trusted GitHub repositories for `SKILL.md` files using the `skill-catalog` helper on PATH. The helper wraps the `gh` CLI; any agent with `Bash` access can use it.

## Prerequisites

- `gh` CLI installed (https://cli.github.com/).
- `gh` authenticated: `gh auth login` (or `GITHUB_TOKEN` set in the environment).

If either is missing, every command exits with code `2` and a stderr message naming what's missing. When that happens, mention it to the user **once** ("skill catalog search unavailable — run `gh auth login` to enable") and skip catalog search for the remainder of the current session — the calling workflow proceeds without it.

## Trusted-repos config

The list of repositories to search lives at `~/.claude/skill-repos.json` — a JSON array of strings:

```json
["anthropics/skills", "myorg/my-skills"]
```

If the file is missing the helper falls back to the built-in default `["anthropics/skills"]`. Users customize by creating the file by hand. Each entry is either:

- `<owner>` — search every public `SKILL.md` under that GitHub user/org.
- `<owner>/<repo>` — search `SKILL.md` files only inside that repository.

If the user asks "what's covered?", read `~/.claude/skill-repos.json` directly (or report the built-in default when the file is absent).

## Commands

### `skill-catalog search "<query>"`

Searches every configured repository for `SKILL.md` files matching `<query>` (under the hood: `gh search code <query> --filename SKILL.md`). Prints a JSON array of hits to stdout, ranked per-repo, deduped across repos:

```json
[
  {
    "id": "anthropics/skills:internal-comms/SKILL.md",
    "repo": "anthropics/skills",
    "path": "internal-comms/SKILL.md",
    "snippet": "…matched fragment…"
  }
]
```

Exit code `3` if there are no hits — still prints `[]`. Run **2–4 targeted queries** per Discovery; broader queries return more noise, narrower queries miss adjacent skills.

### `skill-catalog get <id>`

Fetches the full `SKILL.md` body for a hit returned by `search`. The `<id>` is the `id` field from a search result (`<owner>/<repo>:<path>`). Prints raw markdown to stdout. Exit code `3` if the skill no longer exists.

Do not paste raw skill bodies into the conversation — the description from the SKILL frontmatter plus the `id` is enough to recommend a skill to the user.

## Rate limits

`gh search code` is authenticated GitHub code search, capped at 30 requests per minute per token. The helper makes one request per configured repository per `search` call, so 2–4 queries × N repos sits well inside the budget for typical sessions.
