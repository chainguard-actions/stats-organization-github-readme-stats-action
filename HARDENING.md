<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v2.0.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` into shell command strings. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it, bypassing shell quoting. (1) Line 43: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. (2) Line 55: `run: node ${{ github.action_path }}/index.js`. Both should use the `$GITHUB_ACTION_PATH` environment variable instead (e.g. `node "$GITHUB_ACTION_PATH/index.js"`).

Locations:

- `action.yml:43`
- `action.yml:55`

### github-env-injection (severity: high)

The `run:` block at line 43 writes a value derived from `${{ github.action_path }}` (a `github.*` context) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The offending line is: `echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. A safe fix is to use the `$GITHUB_ACTION_PATH` env var (which avoids template interpolation) and sanitize before writing: `safe=$(printf '%s' "$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/package.json")" | tr -d '\n\r'); echo "package_json=$safe" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two script-injection issues and one github-env-injection issue in action.yml:
1. Line 43 (script-injection + github-env-injection): Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` in the realpath command, converted to a multi-line run block, and added sanitization with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.
2. Line 55 (script-injection): Replaced `node ${{ github.action_path }}/index.js` with `node "$GITHUB_ACTION_PATH/index.js"`, using the built-in environment variable with proper quoting to avoid template engine interpolation.

