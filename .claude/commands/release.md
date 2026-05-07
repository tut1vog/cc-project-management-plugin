Release the plugin: bump the version in `.claude-plugin/plugin.json`, run pre-release checks, commit, tag, and push.

## Arguments

`$ARGUMENTS` — optional: `major`, `minor`, or `patch`. If provided, overrides auto-detection. If omitted, bump type is inferred from the diff since the last tag.

## Steps

### 1. Determine bump type

If `$ARGUMENTS` is `major`, `minor`, or `patch`, use it directly.

Otherwise auto-detect:
- Run `git describe --tags --abbrev=0` to get the last tag.
- Run `git diff <last-tag>..HEAD --name-status -- agents/ skills/` to inspect changes.
- If any top-level entry under `agents/` or `skills/` was added, deleted, or renamed → **MINOR**.
- Otherwise → **PATCH**.

### 2. Compute new version

Read `version` from `.claude-plugin/plugin.json`. Parse as `<major>.<minor>.<patch>`. Apply the bump to get the new version string.

### 3. Update `plugin.json`

Edit `.claude-plugin/plugin.json`: set `"version"` to the new version string.

### 4. Pre-release checks

Run both commands. If either fails, stop and report the error before proceeding:

```
python3 -m json.tool .claude-plugin/plugin.json
python3 -m json.tool .claude-plugin/marketplace.json
```

### 5. Commit

```
git add .claude-plugin/plugin.json
git commit -m "chore: bump plugin.json to v<new-version>"
```

Follow the commit rules in `.claude/rules/git.md`.

### 6. Confirm, then tag and push

Show the user: "About to tag v`<new-version>` and push to origin. Proceed?"

On confirmation:

```
git tag v<new-version>
git push origin main --tags
```
