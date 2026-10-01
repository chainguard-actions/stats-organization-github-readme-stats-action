<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v2.0.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` (a `github.*` context expression) inside shell command strings. Per the script-injection check, ANY `${{ ... }}` expression inside a `run:` block is a violation — including `runner.*` and `github.*` contexts — because the value flows through YAML template substitution before the shell ever sees it.

(a) `resolve_path` step: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"` — sub-rule (a): direct expression interpolation in run block.

(b) `generate` step: `run: node ${{ github.action_path }}/index.js` — sub-rule (a): direct expression interpolation in run block.

Fix: use the `$GITHUB_ACTION_PATH` environment variable instead, e.g. `run: node "$GITHUB_ACTION_PATH/index.js"`

Locations:

- `action.yml:44`
- `action.yml:63`

### github-env-injection (severity: high)

The `resolve_path` step writes a value derived from `${{ github.action_path }}` (a `github.*` context) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The command is: `echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. Although `github.action_path` is not typically attacker-controlled, it is a `github.*` context value and the check requires sanitization before any write to special environment files. Fix: capture the path into a variable, sanitize with `printf '%s' "$val" | tr -d '\n\r'`, then write to GITHUB_OUTPUT. Alternatively, use the `$GITHUB_ACTION_PATH` env var and sanitize before writing.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed both findings in action.yml: (1) Replaced `${{ github.action_path }}` with the `$GITHUB_ACTION_PATH` environment variable in the `resolve_path` step (line 44) and the `generate` step (line 63), eliminating the script-injection risk. (2) In the `resolve_path` step, the computed path is now captured into a variable and sanitized with `printf '%s' "$val" | tr -d '\n\r'` before being written to `$GITHUB_OUTPUT`, fixing the github-env-injection finding.

