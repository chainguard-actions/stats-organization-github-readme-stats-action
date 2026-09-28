<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v2.0.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` — a GitHub Actions expression — inside shell command strings (sub-rule a). Even though `github.action_path` is GitHub-infrastructure-controlled rather than attacker-supplied, any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection violation because the value flows through YAML template substitution before the shell ever sees it, bypassing shell quoting. (1) Line 35: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. (2) Line 57: `run: node ${{ github.action_path }}/index.js`. Fix: replace both with the environment-variable form — set `ACTION_PATH: ${{ github.action_path }}` in an `env:` block and reference `"$ACTION_PATH"` (double-quoted) inside the shell script.

Locations:

- `action.yml:35`
- `action.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in hardened/action/action.yml:
1. Line 35 (Compute workspace-relative package.json path step): Added `env: ACTION_PATH: ${{ github.action_path }}` and replaced the inline `${{ github.action_path }}` expression with `"$ACTION_PATH"` in the shell script.
2. Line 57 (Generate card step): Added `ACTION_PATH: ${{ github.action_path }}` to the existing `env:` block and replaced `node ${{ github.action_path }}/index.js` with `node "$ACTION_PATH/index.js"` (with proper double-quoting).
Both fixes move the GitHub Actions expression out of the shell string and into the env: block, preventing YAML template substitution from bypassing shell quoting.

