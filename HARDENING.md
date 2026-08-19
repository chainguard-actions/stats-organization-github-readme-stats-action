<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block on line 31 of action.yml directly interpolates `${{ github.action_path }}` inside the shell command string. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it. The offending line is: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`. Fix: assign `github.action_path` to an env var and reference it as `"$ACTION_PATH"` in the shell script.

Locations:

- `action.yml:31`

### script-injection (severity: high)

Sub-rule (a): The `run:` block on line 43 of action.yml directly interpolates `${{ github.action_path }}` inside the shell command string. The offending line is: `run: node ${{ github.action_path }}/index.js`. Fix: assign `github.action_path` to an env var (e.g. `ACTION_PATH: ${{ github.action_path }}`) and reference it as `"$ACTION_PATH"` in the shell script.

Locations:

- `action.yml:43`

### github-env-injection (severity: high)

The `run:` block on line 31 of action.yml writes a value derived from the `${{ github.action_path }}` expression to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The command `echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"` passes the realpath output directly into GITHUB_OUTPUT. A path containing newline characters could inject additional key=value pairs into the output file. Fix: capture the value, sanitize it with `printf '%s' "$val" | tr -d '\n\r'`, then write to GITHUB_OUTPUT.

Locations:

- `action.yml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. Line 31 script-injection: Moved `${{ github.action_path }}` into `env: ACTION_PATH: ${{ github.action_path }}` and referenced it as `$ACTION_PATH` in the shell script.
2. Line 31 github-env-injection: Captured the realpath output into a variable, sanitized it with `printf '%s' "$val" | tr -d '\n\r'`, then wrote the sanitized value to $GITHUB_OUTPUT.
3. Line 43 script-injection: Moved `${{ github.action_path }}` into `env: ACTION_PATH: ${{ github.action_path }}` and referenced it as `"$ACTION_PATH"` in the shell script.

