<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v2.0.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` (a `github.*` context expression) inside shell command strings, violating sub-rule (a). Any `${{ ... }}` expression inside a `run:` block is a script-injection risk regardless of context.

Line 44: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`

Line 63: `run: node ${{ github.action_path }}/index.js`

Fix: replace `${{ github.action_path }}` with the pre-set env var `$GITHUB_ACTION_PATH` (which GitHub Actions already exposes as a safe shell variable) or assign it to an `env:` variable and reference that instead.

Locations:

- `action.yml:44`
- `action.yml:63`

### github-env-injection (severity: high)

The `run:` block on line 44 of action.yml writes a value derived from `${{ github.action_path }}` (a `github.*` context) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Although `github.action_path` is not typically attacker-controlled, the check rules classify all `github.*` values as untrusted inputs that must be sanitized before being written to special environment files.

Offending line: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`

Fix: use `$GITHUB_ACTION_PATH` env var directly (no expression interpolation needed) and sanitize before writing: `safe=$(printf '%s' "$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/package.json")" | tr -d '\n\r'); echo "package_json=$safe" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two findings in hardened/action/action.yml:
1. Line 44 (script-injection + github-env-injection): Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` (the pre-set shell env var) and added sanitization via `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.
2. Line 63 (script-injection): Replaced `node ${{ github.action_path }}/index.js` with `node "$GITHUB_ACTION_PATH/index.js"` to eliminate the template expression from the shell command string.

The `working-directory: ${{ github.action_path }}` YAML key (not a run: shell string) was not flagged and left unchanged.

