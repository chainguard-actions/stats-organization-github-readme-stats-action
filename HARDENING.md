<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **stats-organization--github-readme-stats-action/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` (a `github.*` context expression) inside shell command strings, violating sub-rule (a). Although `github.action_path` is typically trusted, ANY `${{ ... }}` expression directly inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting.

1. Line 27: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"` — `${{ github.action_path }}` interpolated directly in the shell command.
2. Line 40: `run: node ${{ github.action_path }}/index.js` — `${{ github.action_path }}` interpolated directly in the shell command.

Fix: use the `$GITHUB_ACTION_PATH` environment variable instead (e.g., `run: node "$GITHUB_ACTION_PATH/index.js"`)

Locations:

- `action.yml:27`
- `action.yml:40`

### github-env-injection (severity: high)

Line 27 writes a value derived from `${{ github.action_path }}` — a `github.*` context expression — directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The full command is: `echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. If `github.action_path` contained newline characters, an attacker could inject additional key=value pairs into GITHUB_OUTPUT. Fix: sanitize the value before writing, or use the `$GITHUB_ACTION_PATH` env var and apply `tr -d '\n\r'` before the write.

Locations:

- `action.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two findings in action.yml: (1) Line 27: replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` env var and added sanitization (`printf '%s' "$raw" | tr -d '\n\r'`) before writing to $GITHUB_OUTPUT to prevent newline injection. (2) Line 40: replaced `node ${{ github.action_path }}/index.js` with `node "$GITHUB_ACTION_PATH/index.js"` using the built-in environment variable with proper shell quoting.

