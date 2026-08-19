<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v1.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): `${{ github.action_path }}` is directly interpolated inside a `run:` shell command string. Before the shell executes the command, GitHub Actions substitutes the expression value into the script text, which can allow injection of shell metacharacters. Offending line: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:27`

### script-injection (severity: high)

Rule (a): `${{ github.action_path }}` is directly interpolated inside a `run:` shell command string. Before the shell executes the command, GitHub Actions substitutes the expression value into the script text, which can allow injection of shell metacharacters. Offending line: `run: node ${{ github.action_path }}/index.js`

Locations:

- `action.yml:43`

### github-env-injection (severity: high)

The `run:` block writes a value derived from `${{ github.action_path }}` (a `github.*` context expression) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A newline embedded in the value could inject additional key=value pairs into the output file. Offending line: `echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three findings in hardened/action/action.yml:
1. (line 27) script-injection: Moved `${{ github.action_path }}` into an `env:` block as `ACTION_PATH` in the 'Resolve relative path to package.json' step; the shell script now references `$ACTION_PATH` instead of the raw expression.
2. (line 27) github-env-injection: Added sanitization of the realpath output using `printf '%s' "$raw" | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`.
3. (line 43) script-injection: Moved `${{ github.action_path }}` into an `env:` block as `ACTION_PATH` in the 'Generate card' step; the shell script now uses `"$ACTION_PATH/index.js"` instead of the raw expression.

