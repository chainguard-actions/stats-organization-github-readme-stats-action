<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v1.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action uses `actions/setup-node@v6`, which is pinned to a mutable tag (`v6`) rather than a full 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling supply-chain attacks. It should be replaced with a pinned SHA, e.g. `actions/setup-node@<40-char-sha> # v6`.

Locations:

- `action.yml:27`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. Line 35 contains: `run: node ${{ github.action_path }}/index.js`. GitHub Actions performs template substitution before the shell ever sees the string, so any expression — including `github.action_path` — is expanded as raw text into the shell command. This is a script-injection risk. The safe alternative is to pass the value via an environment variable and reference it as `$GITHUB_ACTION_PATH` (which is already set by the runner), e.g. `run: node "$GITHUB_ACTION_PATH/index.js"`.

Locations:

- `action.yml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned actions/setup-node from mutable tag @v6 to full commit SHA @249970729cb0ef3589644e2896645e5dc5ba9c38 # v6. 2. Replaced `node ${{ github.action_path }}/index.js` with `node "$GITHUB_ACTION_PATH/index.js"` — the runner already exposes GITHUB_ACTION_PATH as an environment variable, eliminating the template-injection risk in the shell command.

