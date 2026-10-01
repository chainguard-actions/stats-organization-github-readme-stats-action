<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Resolve relative path to package.json' run: block directly interpolates the GitHub Actions expression `${{ github.action_path }}` inside a shell command string. Any `${{ ... }}` expression inside a run: block is a script-injection risk because the value flows through YAML template substitution before the shell ever sees it. Offending line: `run: echo "package_json=$(realpath --relative-to="$GITHUB_WORKSPACE" "${{ github.action_path }}/package.json")" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:32`

### script-injection (severity: high)

Sub-rule (a): The 'Generate card' run: block directly interpolates the GitHub Actions expression `${{ github.action_path }}` inside a shell command string. Any `${{ ... }}` expression inside a run: block is a script-injection risk because the value flows through YAML template substitution before the shell ever sees it. Offending line: `run: node ${{ github.action_path }}/index.js`

Locations:

- `action.yml:47`

### github-env-injection (severity: high)

The 'Resolve relative path to package.json' run: block writes a value derived from `${{ github.action_path }}` (a github.* context, which is treated as untrusted per the check rules) directly to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The safe pattern would be to capture the path into a variable, sanitize it with `printf '%s' "$VAR" | tr -d '\n\r'`, and then write the sanitized value to $GITHUB_OUTPUT.

Locations:

- `action.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three findings in action.yml:
1. 'Resolve relative path to package.json' step (line 32): Moved `${{ github.action_path }}` into the `env:` block as `ACTION_PATH`, referenced it as `$ACTION_PATH` in the shell script, and added sanitization (`printf '%s' "$safe_path" | tr -d '\n\r'`) before writing to `$GITHUB_OUTPUT`.
2. 'Generate card' step (line 47): Moved `${{ github.action_path }}` into the `env:` block as `ACTION_PATH` and referenced it as `"$ACTION_PATH/index.js"` in the shell command.

