<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` expressions inside shell command strings (rule a). Although `github.action_path` is not attacker-controlled via PRs, any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection finding per the check rules.

1. Step 'Compute workspace-relative package.json path': `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"` — `${{ github.action_path }}` is interpolated directly in the shell command.

2. Step 'Generate card': `run: node ${{ github.action_path }}/index.js` — `${{ github.action_path }}` is interpolated directly in the shell command.

Fix: use the `$GITHUB_ACTION_PATH` environment variable instead of `${{ github.action_path }}` in `run:` blocks.

Locations:

- `action.yml:36`
- `action.yml:51`

### github-env-injection (severity: high)

The 'Compute workspace-relative package.json path' step writes a value derived from `${{ github.action_path }}` directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The command is: `echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. The expression `${{ github.action_path }}` is substituted by the Actions runner before the shell executes, so a newline embedded in the value could inject additional key=value pairs into `$GITHUB_OUTPUT`.

Fix: use `$GITHUB_ACTION_PATH` env var and sanitize before writing: `safe=$(printf '%s' "$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/package.json")" | tr -d '\n\r'); echo "package_json=$safe" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:36`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed both findings in action.yml:
1. script-injection (lines 36 and 51): Replaced `${{ github.action_path }}` with the `$GITHUB_ACTION_PATH` environment variable in both `run:` blocks. The 'Generate card' step now uses `node "$GITHUB_ACTION_PATH/index.js"` (properly quoted).
2. github-env-injection (line 36): The 'Compute workspace-relative package.json path' step now sanitizes the computed path with `printf '%s' ... | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`, preventing newline injection. Both fixes together eliminate the direct `${{ github.action_path }}` interpolation in shell command strings.

