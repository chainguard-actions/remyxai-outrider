<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.11** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml pin to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`. These should be pinned to their full SHA digests (e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`).

Locations:

- `action.yml:285`
- `action.yml:290`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks directly interpolate `${{ github.action_path }}` inside shell command strings. Any `${{ ... }}` expression interpolated directly in a `run:` script is a script-injection risk because the value is substituted by the GitHub Actions template engine before the shell ever sees it, bypassing shell quoting. Offending lines: (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the "Install gh-graph selection-pass tool on PATH" step; (2) `python ${{ github.action_path }}/src/run.py` in the "Recommend + implement + open PR" step. Fix: use the `$GITHUB_ACTION_PATH` environment variable instead, which is already set by the runner and does not require template interpolation.

Locations:

- `action.yml:310`
- `action.yml:430`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all four issues in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065; (2) Pinned actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020; (3) Replaced ${{ github.action_path }} with $GITHUB_ACTION_PATH in the 'Install gh-graph selection-pass tool on PATH' step; (4) Replaced ${{ github.action_path }} with $GITHUB_ACTION_PATH (with added quotes) in the 'Recommend + implement + open PR' step. The $GITHUB_ACTION_PATH environment variable is set by the GitHub Actions runner and avoids template interpolation into the shell script.

