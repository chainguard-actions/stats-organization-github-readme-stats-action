<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v2.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` (a `github.*` context expression) into shell command strings. Per the check rules, ANY `${{ ... }}` expression inside a `run:` block is a script-injection finding. (1) Line 55: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. (2) Line 68: `run: node ${{ github.action_path }}/index.js`. These should be replaced with the `$GITHUB_ACTION_PATH` environment variable, which is already available as a safe shell variable.

Locations:

- `action.yml:55`
- `action.yml:68`

### github-env-injection (severity: high)

A `run:` block writes a value derived from `${{ github.action_path }}` (a `github.*` context) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Line 55: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. The `github.action_path` value is embedded in the command and its output is written unsanitized to GITHUB_OUTPUT.

Locations:

- `action.yml:55`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two locations in action.yml: (1) Line 55: Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` in the run: block and added sanitization (`tr -d '\n\r'`) before writing to $GITHUB_OUTPUT, restructured as a multi-line run block. (2) Line 68: Replaced `${{ github.action_path }}` with `"$GITHUB_ACTION_PATH"` (properly quoted) in the node command. Both fixes use the safe `$GITHUB_ACTION_PATH` shell variable that GitHub Actions provides automatically, eliminating the script-injection and github-env-injection risks.

