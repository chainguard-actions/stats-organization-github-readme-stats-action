<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings (sub-rule a). Any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it, bypassing shell quoting.

1. Line 27: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"` — `${{ github.action_path }}` is interpolated directly in the shell command.

2. Line 38: `run: node ${{ github.action_path }}/index.js` — `${{ github.action_path }}` is interpolated directly in the shell command.

Fix: use the `$GITHUB_ACTION_PATH` environment variable (automatically set by the runner) instead of `${{ github.action_path }}` inside `run:` blocks.

Locations:

- `action.yml:27`
- `action.yml:38`

### github-env-injection (severity: high)

The `run:` block on line 27 writes a value derived from `${{ github.action_path }}` directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The expression is interpolated by the Actions template engine before the shell runs, so a newline embedded in the value could inject additional key=value pairs into the output file.

Offending line: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`

Fix: replace `${{ github.action_path }}` with the `$GITHUB_ACTION_PATH` env var and apply sanitization before writing to `$GITHUB_OUTPUT`, e.g.:
```bash
safe_path=$(printf '%s' "$GITHUB_ACTION_PATH" | tr -d '\n\r')
echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "$safe_path/package.json")" >> "$GITHUB_OUTPUT"
```

Locations:

- `action.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed both findings in hardened/action/action.yml:
1. Line 27 (script-injection + github-env-injection): Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` (runner-provided env var) and added sanitization via `printf '%s' "$GITHUB_ACTION_PATH" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.
2. Line 38 (script-injection): Replaced `node ${{ github.action_path }}/index.js` with `node "$GITHUB_ACTION_PATH/index.js"`, using the runner-provided environment variable instead of the template expression.

