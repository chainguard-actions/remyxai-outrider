<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.6.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it. (1) In the 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) In the 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py`. Both should use the `$GITHUB_ACTION_PATH` environment variable instead, which is already set by the runner and does not go through template substitution.

Locations:

- `action.yml:202`
- `action.yml:236`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or compromised. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`. Each should be replaced with the full SHA digest of the intended release, e.g. `actions/setup-python@<40-char-sha> # v5`.

Locations:

- `action.yml:159`
- `action.yml:163`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all four issues in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to full SHA a26af69be951a213d495a4c3e4e4022e16d87065. (2) Pinned actions/setup-node@v4 to full SHA 49933ea5288caeca8642d1e84afbd3f7d6820020. (3) Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` in the 'Install gh-graph' step (line 202). (4) Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` in the 'Recommend + implement + open PR' step (line 236), also adding proper double-quoting around the path.

