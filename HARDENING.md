<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings (sub-rule a: any `${{ ... }}` expression in a `run:` block is a script-injection risk). Line 27: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. Line 41: `run: node ${{ github.action_path }}/index.js`. These should be replaced with the `$GITHUB_ACTION_PATH` environment variable instead of the template expression.

Locations:

- `action.yml:27`
- `action.yml:41`

### github-env-injection (severity: high)

Line 27 of action.yml writes a value derived from `${{ github.action_path }}` directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The run block is: `echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. The expression is interpolated directly into the shell command and then written to the special environment file, bypassing newline sanitization.

Locations:

- `action.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed both findings in action.yml:
1. Line 27 (script-injection + github-env-injection): Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` and rewrote the run block to sanitize the value with `printf '%s' "$raw" | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`.
2. Line 41 (script-injection): Replaced `${{ github.action_path }}` with `"$GITHUB_ACTION_PATH"` (properly quoted shell variable).

