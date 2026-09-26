<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v2.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` (a `github.*` context expression) into shell command strings, violating sub-rule (a). Any `${{ ... }}` expression inside a `run:` block is a script-injection risk regardless of which context it reads from.

1. Line 29: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"` — `${{ github.action_path }}` is interpolated directly into the shell command.

2. Line 43: `run: node ${{ github.action_path }}/index.js` — `${{ github.action_path }}` is interpolated directly into the shell command.

Fix: Move `github.action_path` into an `env:` variable and reference it as a quoted shell variable (e.g., `"$ACTION_PATH"`) inside the `run:` block.

Locations:

- `action.yml:29`
- `action.yml:43`

### github-env-injection (severity: high)

Line 29 of action.yml writes a value derived from `${{ github.action_path }}` (a `github.*` context value) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

Offending line: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`

Fix: Assign `github.action_path` to an env var, sanitize it with `safe=$(printf '%s' "$ACTION_PATH" | tr -d '\n\r')`, then use the sanitized value in the echo command before writing to `$GITHUB_OUTPUT`.

Locations:

- `action.yml:29`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed both findings in action.yml:
1. Line 29 (script-injection + github-env-injection): Moved `${{ github.action_path }}` into an `env:` block as `ACTION_PATH`, added sanitization via `printf '%s' "$ACTION_PATH" | tr -d '\n\r'` into `safe_action_path`, and used the sanitized variable in the `realpath` command before writing to `$GITHUB_OUTPUT`.
2. Line 43 (script-injection): Moved `${{ github.action_path }}` into the step's `env:` block as `ACTION_PATH` and referenced it as `"$ACTION_PATH/index.js"` in the `run:` block.

