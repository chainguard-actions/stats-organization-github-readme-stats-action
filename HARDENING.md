<!-- markdownlint-disable -->

# Hardening Report: stats-organization--github-readme-stats-action/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **stats-organization--github-readme-stats-action/v1.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a `run:` shell command in action.yml. The line `run: node ${{ github.action_path }}/index.js` embeds `${{ github.action_path }}` directly into the shell command string. Per the check rules, ANY `${{ ... }}` expression directly inside a `run:` block is a script-injection finding regardless of which context it reads from. The value should instead be referenced via an environment variable (e.g., `env: ACTION_PATH: ${{ github.action_path }}`) and then used as `"$ACTION_PATH"` in the run script.

Locations:

- `action.yml:24`

### unpinned-uses (severity: high)

Workflow file e2e.yml references actions using mutable tag refs instead of full 40-character SHA commit hashes, making the workflow vulnerable to supply-chain attacks if the tag is moved. Failing references: `actions/checkout@v4` and `actions/setup-node@v4`.

Locations:

- `.github/workflows/e2e.yml:12`
- `.github/workflows/e2e.yml:13`

### unpinned-uses (severity: high)

Workflow file release.yml references actions using mutable tag refs instead of full 40-character SHA commit hashes, making the workflow vulnerable to supply-chain attacks if the tags are moved. Failing references: `actions/checkout@v4`, `haya14busa/action-update-semver@v1`, and `softprops/action-gh-release@v1`.

Locations:

- `.github/workflows/release.yml:11`
- `.github/workflows/release.yml:12`
- `.github/workflows/release.yml:14`

### missing-permissions (severity: medium)

Workflow file e2e.yml has no top-level `permissions:` key and the single job `e2e` also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad. A minimal permissions block (e.g., `permissions: contents: read`) should be added.

Locations:

- `.github/workflows/e2e.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed 4 findings across 3 files: (1) action.yml: moved ${{ github.action_path }} from the run: shell command into an env: block as ACTION_PATH, referenced as "$ACTION_PATH/index.js" in the shell script to prevent script injection; (2) e2e.yml: pinned actions/checkout@v4 to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 and actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, and added top-level permissions: contents: read; (3) release.yml: pinned actions/checkout@v4 to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5, haya14busa/action-update-semver@v1 to SHA 7d2c558640ea49e798d46539536190aff8c18715, and softprops/action-gh-release@v1 to SHA de2c0eb89ae2a093876385947365aca7b0e5f844. The release.yml already had permissions: contents: write which is appropriate for creating releases.

