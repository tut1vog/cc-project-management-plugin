---
paths:
  - .claude-plugin/plugin.json
  - .claude-plugin/marketplace.json
---

# Releasing

## Rules
- Bump **only** `.claude-plugin/plugin.json` `version` when releasing.
- Version scheme: [SemVer](https://semver.org/). MINOR when a top-level entry under `agents/` or `skills/` is added, removed, or renamed; PATCH for everything else; MAJOR only on explicit request.
- Pre-release checks (all must pass):
  1. `python3 -m json.tool .claude-plugin/plugin.json`
  2. `python3 -m json.tool .claude-plugin/marketplace.json`
  3. `claude --plugin-dir ./` — all expected agents appear in `/agents`.
- Release procedure:
  1. Update `plugin.json` `version`, commit: `chore: bump plugin.json to v<x.y.z>`.
  2. `git tag v<x.y.z>` then `git push origin main --tags` (both require user confirmation).

## Example

```
# edit .claude-plugin/plugin.json: "version": "0.2.0"
git add .claude-plugin/plugin.json
git commit -m "chore: bump plugin.json to v0.2.0"
python3 -m json.tool .claude-plugin/plugin.json
python3 -m json.tool .claude-plugin/marketplace.json
claude --plugin-dir ./
git tag v0.2.0
git push origin main --tags
```
