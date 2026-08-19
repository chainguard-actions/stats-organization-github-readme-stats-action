<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` — a GitHub Actions expression — inside shell command strings (sub-rule a). Any `${{ ... }}` expression interpolated directly into a `run:` block flows through YAML template substitution before the shell sees it, enabling script injection if the value contains shell metacharacters.

1. Line 36: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"` — `${{ github.action_path }}` is interpolated directly in the shell command.

2. Line 55: `run: node ${{ github.action_path }}/index.js` — `${{ github.action_path }}` is interpolated directly in the shell command.

Fix: use the pre-set `$GITHUB_ACTION_PATH` environment variable instead of the `${{ github.action_path }}` expression, e.g. `run: node "$GITHUB_ACTION_PATH/index.js"`.

Locations:

- `action.yml:36`
- `action.yml:55`

### github-env-injection (severity: high)

Line 36 of action.yml writes a value derived from `${{ github.action_path }}` (a `github.*` context — an untrusted-input source per the check rules) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The offending line is:

`run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`

Fix: assign `$GITHUB_ACTION_PATH` to a local variable, sanitize it with `printf '%s' "$GITHUB_ACTION_PATH" | tr -d '\n\r'`, then use the sanitized value in the write to `$GITHUB_OUTPUT`.

Locations:

- `action.yml:36`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed both findings in action.yml:
1. Line 36 (script-injection + github-env-injection): Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` and added sanitization (`printf '%s' "$GITHUB_ACTION_PATH" | tr -d '\n\r'`) before writing to `$GITHUB_OUTPUT`.
2. Line 55 (script-injection): Replaced `node ${{ github.action_path }}/index.js` with `node "$GITHUB_ACTION_PATH/index.js"` using the pre-set environment variable instead of the template expression.

