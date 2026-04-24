# cc-project-management-plugin

A Claude Code plugin bundling three orchestration subagents (`initializer`, `advisor`, `director`) plus the `skillex-mcp` server, distributed as a single-plugin marketplace so users can install it globally with `/plugin install` instead of copying `.md` files into every project.

## Stack
- Language / runtime: **none at build time** — the repo is Markdown (agent definitions, rules, docs) and JSON (plugin manifests, MCP config).
- Runtime dependency on consumer machines: **Node ≥ 20** (required by `skillex-mcp`, launched via `npx -y github:tut1vog/skillex-mcp`).
- No compile step, no test suite, no dependency lockfiles.

## Directory Layout
```
.
├── .claude-plugin/
│   ├── plugin.json         # plugin manifest (name, version, metadata)
│   └── marketplace.json    # self-referential single-plugin marketplace catalog
├── agents/                 # subagent definitions — one .md per agent (plugin spec location)
│   ├── initializer.md
│   ├── advisor.md
│   └── director.md
├── .mcp.json               # skillex-mcp wiring (default SKILLS_MCP_REPOS=anthropics/skills)
├── .claude/
│   └── rules/              # behavioral rules for working on THIS repo, loaded on demand
├── docs/                   # human-facing per-agent documentation
├── CLAUDE.md               # this file
├── README.md               # consumer-facing install + usage guide
├── LICENSE                 # MIT
└── project-brief.md        # planning input for director
```

Note: `agents/` lives at the **plugin root**, not under `.claude/agents/`. That nested location is how consumers see agents after installation — the plugin spec requires them at the root, and `.claude-plugin/` holds only `plugin.json` + `marketplace.json`.

## Canonical Commands
- Local plugin test:       `claude --plugin-dir ./`
- Reload without restart:  `/reload-plugins` (inside a Claude Code session)
- Validate JSON manifest:  `python3 -m json.tool .claude-plugin/plugin.json`
- Validate marketplace:    `python3 -m json.tool .claude-plugin/marketplace.json`
- Validate MCP config:     `python3 -m json.tool .mcp.json`
- Smoke-test skillex MCP:  `npx -y github:tut1vog/skillex-mcp --help`  *(first run clones + builds skillex-mcp; subsequent runs cached)*
- Release (tag + push):    `git tag v<semver> && git push origin main --tags`

There is no `build`, `test`, `lint`, or `run` target — Markdown + JSON only.

## Rules (load on demand)
Each rule file below is a focused behavioral contract. Read a rule file when its trigger matches your task — do not auto-load.

- `.claude/rules/git.md` — read before making any commit or tag
- `.claude/rules/releasing.md` — read before bumping the plugin version or cutting a release
- `.claude/rules/agent-authoring.md` — read before creating or editing any file under `agents/`
- `.claude/rules/mcp-config.md` — read before editing `.mcp.json` or changing MCP server configuration

## Planning Context
For current intent, scope, and how director should operate on this repo, see `project-brief.md`. Director permissions are managed locally by the maintainer in `.claude/settings.local.json` (gitignored) — there is no committed `.claude/settings.json`.
