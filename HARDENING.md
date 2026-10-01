<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v1.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses `actions/setup-node@v6`, which is a mutable version tag rather than a pinned 40-character SHA commit hash. This means the action could silently pull in a different (potentially malicious) version of the dependency if the tag is moved. It should be pinned to a full SHA, e.g. `actions/setup-node@1d0ff469b18977b4dc9f2d0c8e5c4b7e5e5e5e5e # v6`.

Locations:

- `action.yml:27`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is directly interpolated inside a `run:` shell command string. Line 35 contains `run: node ${{ github.action_path }}/index.js` — the expression `${{ github.action_path }}` is substituted by the Actions runner before the shell ever sees the command, meaning any special characters in the value are not quoted and could alter the shell command. The safe alternative is to use the pre-set environment variable `$GITHUB_ACTION_PATH` instead: `run: node "$GITHUB_ACTION_PATH/index.js"`.

Locations:

- `action.yml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned actions/setup-node from mutable tag @v6 to full SHA @249970729cb0ef3589644e2896645e5dc5ba9c38 with a # v6 comment for readability. 2. Replaced `${{ github.action_path }}` expression inside the `run:` shell command with the pre-set `$GITHUB_ACTION_PATH` environment variable, using proper quoting: `node "$GITHUB_ACTION_PATH/index.js"`. The `working-directory: ${{ github.action_path }}` usage in the Install dependencies step is a YAML field (not a shell command), so it is not subject to shell injection and was left unchanged.

