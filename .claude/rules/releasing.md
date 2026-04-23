# Releasing

**When to read this**: read before bumping the plugin version or cutting a release.

## Rules
- Bump **only** `.claude-plugin/plugin.json` `version` when releasing. `.claude-plugin/marketplace.json` has no per-plugin version field in the current manifest — if a `version` is added to the marketplace plugin entry later, it must match `plugin.json` exactly.
- Version scheme: [SemVer](https://semver.org/). Pre-1.0 releases bump MINOR for any user-visible change; bump PATCH only for documentation / metadata fixes that don't change what the plugin loads.
- Never tag a release if any of the following haven't passed:
  1. `python3 -m json.tool .claude-plugin/plugin.json` — valid JSON.
  2. `python3 -m json.tool .claude-plugin/marketplace.json` — valid JSON.
  3. `python3 -m json.tool .mcp.json` — valid JSON.
  4. `claude --plugin-dir ./` — plugin loads and all three agents appear in `/agents`.
- Release procedure:
  1. Update `plugin.json` `version` to the target, commit with `chore: bump plugin.json to v<x.y.z>`.
  2. Tag: `git tag v<x.y.z>` on that commit.
  3. Push: `git push origin main --tags`.
- `git push` and `git tag` both require user confirmation on this repo (permissions live in `.claude/settings.local.json`). Do not autonomously push release tags.
- Treat a change to `.mcp.json`'s `args` (e.g. switching from github source to npm) as a **MINOR** bump — it changes what the plugin executes on consumer machines.
- Treat a change to default `SKILLS_MCP_REPOS` as a **MINOR** bump — it changes what users see by default.

## Examples

Valid release flow for a minor bump:
```
# edit .claude-plugin/plugin.json: "version": "0.2.0"
git add .claude-plugin/plugin.json
git commit -m "chore: bump plugin.json to v0.2.0"
# verify
python3 -m json.tool .claude-plugin/plugin.json
python3 -m json.tool .mcp.json
claude --plugin-dir ./  # manual check: /agents lists all three
# tag
git tag v0.2.0
# push (prompts for confirmation — this is a public repo)
git push origin main --tags
```

Invalid — tag doesn't match manifest:
```
# plugin.json still says "version": "0.1.0"
git tag v0.2.0   # WRONG — tag and manifest must match
```
