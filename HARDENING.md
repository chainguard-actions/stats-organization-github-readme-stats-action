<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v2.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` (a GitHub Actions expression) inside shell command strings. Per rule (a), any `${{ ... }}` expression interpolated directly inside a `run:` block is a script-injection risk, as the value flows through YAML template substitution before the shell processes it. (1) `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"` — the expression is embedded directly in the shell command. (2) `run: node ${{ github.action_path }}/index.js` — same issue. Both should use the `$GITHUB_ACTION_PATH` environment variable instead (e.g. `run: node "$GITHUB_ACTION_PATH/index.js"`)

Locations:

- `action.yml:55`
- `action.yml:68`

### github-env-injection (severity: high)

The 'Compute workspace-relative package.json path' step writes a value derived from `${{ github.action_path }}` (a `github.*` context) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The command is: `echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. A newline injected into `github.action_path` could allow an attacker to inject arbitrary key-value pairs into `$GITHUB_OUTPUT`. The fix is to use the pre-set `$GITHUB_ACTION_PATH` environment variable (which avoids template injection entirely) and sanitize before writing: `safe=$(printf '%s' "$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/package.json")" | tr -d '\n\r'); echo "package_json=$safe" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:55`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed both findings in action.yml:
1. Line 55 (script-injection + github-env-injection): Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` in the 'Compute workspace-relative package.json path' step's run: block, and added sanitization (`printf '%s' ... | tr -d '\n\r'`) before writing to $GITHUB_OUTPUT.
2. Line 68 (script-injection): Replaced `node ${{ github.action_path }}/index.js` with `node "$GITHUB_ACTION_PATH/index.js"` in the 'Generate card' step's run: block.

The `working-directory: ${{ github.action_path }}` field in the 'Install dependencies' step is a YAML `with:` field (not a `run:` shell string), so it is not a script-injection risk and was left unchanged.

