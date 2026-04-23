# MCP configuration

**When to read this**: read before editing `.mcp.json` or changing MCP server configuration.

## Rules
- The plugin ships exactly one MCP server: `skillex`, sourced from `github:tut1vog/skillex-mcp` via `npx -y`.
- **Source is GitHub, not npm.** `skillex-mcp` is not published to the npm registry. `npx -y github:tut1vog/skillex-mcp` relies on the `prepare` script in `skillex-mcp`'s `package.json` to run `tsc` and produce `dist/index.js` at install time. If that repo ever removes its `prepare` script or drops `devDependencies.typescript`, this plugin's MCP server breaks. Do not switch to `npx -y skillex-mcp` until the package is actually on npm.
- `SKILLS_MCP_REPOS` is **comma-separated** and whatever a user sets **replaces** the default — skillex has no append semantics. The plugin ships `"SKILLS_MCP_REPOS": "anthropics/skills"` as the baked-in default; document in `README.md` how users override/extend by setting the env var before launching Claude Code.
- Changing the default `SKILLS_MCP_REPOS` is a **MINOR** version bump (see `releasing.md`) — it changes what users see by default.
- Switching the source (github ↔ npm ↔ local) is a **MINOR** version bump.
- Keep `.mcp.json` minimal. Do not add servers the agents don't actively use. Every MCP server launches a subprocess on every Claude Code session; bloat has real cost.
- End users must have **Node ≥ 20** for `npx` to launch the server. Document this in `README.md` prerequisites.
- After any edit to `.mcp.json`:
  1. `python -m json.tool .mcp.json` — validate JSON.
  2. `claude --plugin-dir ./` — confirm the server starts; check `/mcp` inside the session.

## Examples

Current shipped config (correct):
```json
{
  "mcpServers": {
    "skillex": {
      "command": "npx",
      "args": ["-y", "github:tut1vog/skillex-mcp"],
      "env": {
        "SKILLS_MCP_REPOS": "anthropics/skills"
      }
    }
  }
}
```

If/when skillex-mcp is published to npm, update to:
```json
{
  "mcpServers": {
    "skillex": {
      "command": "npx",
      "args": ["-y", "skillex-mcp"],
      "env": {
        "SKILLS_MCP_REPOS": "anthropics/skills"
      }
    }
  }
}
```
and bump `plugin.json` to a new MINOR version.

How a user extends the default repo list (document this in README, not in the manifest):
```bash
# replace the default entirely
SKILLS_MCP_REPOS="anthropics/skills,myorg/my-skills" claude

# or set in shell profile for every session
export SKILLS_MCP_REPOS="anthropics/skills,myorg/my-skills"
```
