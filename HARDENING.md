<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **stats-organization--github-readme-stats-action/v2.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` as a GitHub Actions expression inside shell command strings. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the Actions runner before the shell ever sees it, bypassing shell quoting. (1) Line 31: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. (2) Line 46: `run: node ${{ github.action_path }}/index.js`. Both should use the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:31`
- `action.yml:46`

### github-env-injection (severity: high)

Line 31 writes a value derived from `${{ github.action_path }}` — interpolated directly in the `run:` block — to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The offending line is: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. The fix is to use the `$GITHUB_ACTION_PATH` env var and apply sanitization before writing to `$GITHUB_OUTPUT`.

Locations:

- `action.yml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two findings in action.yml: (1) script-injection on lines 31 and 46 — replaced `${{ github.action_path }}` expressions inside `run:` blocks with the safe `$GITHUB_ACTION_PATH` environment variable; (2) github-env-injection on line 31 — rewrote the step to sanitize the path value with `tr -d '\n\r'` before writing to `$GITHUB_OUTPUT` using `printf`, preventing newline injection. The `node` invocation on the former line 46 now also properly double-quotes the path (`"$GITHUB_ACTION_PATH/index.js"`) for correctness.

