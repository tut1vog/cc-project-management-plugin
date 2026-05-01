# cc-project-management-plugin

A Claude Code plugin bundling three orchestration subagents (`scaffolder`, `director`, `investigator`), the `plan-management` and `project-scaffolding` skills, and the `skillex-mcp` server — distributed as a single-plugin marketplace so users can install it globally with `/plugin install` instead of copying `.md` files into every project.

## Stack
- Language / runtime: **none at build time** — the repo is Markdown (agent definitions, rules, docs), JSON (plugin manifests, MCP config), and one stdlib-only Python helper (`bin/plan-management`).
- Runtime dependencies on consumer machines: **Node ≥ 20** (required by `skillex-mcp`, launched via `npx -y github:tut1vog/skillex-mcp`) and **Python 3** (required by the `plan-management` helper command).
- No compile step, no test suite, no dependency lockfiles.

## Directory Layout
```
.
├── .claude-plugin/
│   ├── plugin.json         # plugin manifest (name, version, metadata)
│   └── marketplace.json    # self-referential single-plugin marketplace catalog
├── agents/                 # subagent definitions — one .md per agent (plugin spec location)
│   ├── scaffolder.md
│   ├── director.md
│   └── investigator.md
├── skills/                 # bundled skills — one dir per skill (plugin spec location)
│   ├── plan-management/
│   │   └── SKILL.md
│   └── project-scaffolding/
│       ├── SKILL.md
│       └── references/     # hybrid rule templates filled from Discovery answers
├── bin/                    # executables added to the Bash tool's PATH while the plugin is enabled
│   └── plan-management     # stdlib Python: walks plan: chains, prints merged plan
├── .mcp.json               # skillex-mcp wiring (default SKILLS_MCP_REPOS=anthropics/skills)
├── .claude/
│   └── rules/              # behavioral rules for working on THIS repo, loaded on demand
├── docs/                   # human-facing per-agent documentation
├── CLAUDE.md               # this file
├── README.md               # consumer-facing install + usage guide
└── LICENSE                 # MIT
```

Note: `agents/` and `skills/` both live at the **plugin root**, not under `.claude/agents/` or `.claude/skills/`. Those nested locations are how consumers see them after installation — the plugin spec requires them at the root, and `.claude-plugin/` holds only `plugin.json` + `marketplace.json`.

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
- `.claude/rules/agent-authoring.md` — read before creating or editing any file under `agents/` or `skills/`
- `.claude/rules/mcp-config.md` — read before editing `.mcp.json` or changing MCP server configuration
- `.claude/rules/no-legacy.md` — read before editing any prose (agent prompts, skill prompts, rules, docs, READMEs)

## Skills (bundled)
- `skills/plan-management/SKILL.md` — canonical format spec and read/write commands for the `plan:` and `Task:` journal commit messages director uses to store plan state in git history. Read it before editing director's plan/journal behavior or any agent that needs to inspect plan state.
- `skills/project-scaffolding/SKILL.md` — canonical spec for the durable files scaffolder writes after Discovery (`CLAUDE.md`, `CLAUDE.local.md`, rule files, `.claude/settings.local.json`) plus the Requirements Summary input contract and reference template catalog. Read it before editing scaffolder's behavior, the file overwrite/git-bootstrap policy, or the rule reference templates.

## Permissions
Director's permissions for this repo live in the maintainer's local `.claude/settings.local.json` (gitignored).
