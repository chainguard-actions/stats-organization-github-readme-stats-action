<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **stats-organization--github-readme-stats-action/v1.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses `actions/setup-node@v6`, which is pinned to a mutable version tag (`@v6`) rather than an immutable 40-character commit SHA. This means the referenced action could be silently replaced with a different (potentially malicious) version without any change to this file.

Locations:

- `action.yml:24`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. The step `Generate card` contains `run: node ${{ github.action_path }}/index.js`. GitHub Actions performs template substitution on `${{ github.action_path }}` before the shell ever sees the string, meaning any unexpected value in that context flows directly into the shell command without quoting or sanitization. The safe alternative is to use the pre-set environment variable `$GITHUB_ACTION_PATH` instead: `run: node "$GITHUB_ACTION_PATH/index.js"`.

Locations:

- `action.yml:29`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned `actions/setup-node@v6` to its full commit SHA `48b55a011bda9f5d6aeb4c2d9c7362e8dae4041e` with a `# v6` comment for readability. 2. Replaced `node ${{ github.action_path }}/index.js` in the 'Generate card' step with `node "$GITHUB_ACTION_PATH/index.js"`, using the pre-set `$GITHUB_ACTION_PATH` environment variable instead of a template expression interpolated directly into the shell command string.

