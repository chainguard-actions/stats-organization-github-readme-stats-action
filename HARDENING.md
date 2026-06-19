<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **stats-organization--github-readme-stats-action/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` (a GitHub Actions expression) inside shell command strings, violating rule (a). Any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection finding. Line 36: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. Line 55: `run: node ${{ github.action_path }}/index.js`. Fix: use the `$GITHUB_ACTION_PATH` environment variable instead of the expression form.

Locations:

- `action.yml:36`
- `action.yml:55`

### github-env-injection (severity: high)

The `run:` block at line 36 writes a value derived from `${{ github.action_path }}` directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The expression is interpolated inline into the shell command that produces the value written to the special environment file. Offending line: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. Fix: replace the expression with the `$GITHUB_ACTION_PATH` env var or apply the sanitization pipeline before writing to `$GITHUB_OUTPUT`.

Locations:

- `action.yml:36`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two script-injection findings and one github-env-injection finding in action.yml:
1. Line 36 (script-injection + github-env-injection): Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` env var, restructured the run block to multi-line format, and added sanitization (`printf '%s' "$raw" | tr -d '\n\r'`) before writing to `$GITHUB_OUTPUT`.
2. Line 55 (script-injection): Replaced `node ${{ github.action_path }}/index.js` with `node "$GITHUB_ACTION_PATH/index.js"`, using the built-in env var and properly quoting the path.

