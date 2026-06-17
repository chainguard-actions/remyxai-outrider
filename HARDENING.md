<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.6.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`.

Locations:

- `action.yml:196`
- `action.yml:200`

### script-injection (severity: high)

Rule (a): Two `run:` blocks directly interpolate `${{ github.action_path }}` inside shell command strings. Any `${{ ... }}` expression interpolated directly into a `run:` script is a script-injection risk because the value flows through YAML template substitution before the shell ever sees it. Offending lines: (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the 'Install gh-graph selection-pass tool on PATH' step; (2) `python ${{ github.action_path }}/src/run.py` in the 'Recommend + implement + open PR' step. The safe fix is to use the `$GITHUB_ACTION_PATH` environment variable (already set by the runner) instead of the `${{ github.action_path }}` expression.

Locations:

- `action.yml:218`
- `action.yml:246`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three issues in action.yml: (1) Pinned actions/setup-python@v5 to immutable SHA a26af69be951a213d495a4c3e4e4022e16d87065; (2) Pinned actions/setup-node@v4 to immutable SHA 49933ea5288caeca8642d1e84afbd3f7d6820020; (3) Replaced both ${{ github.action_path }} template expressions in run: blocks with the $GITHUB_ACTION_PATH environment variable (already set by the runner), eliminating the script-injection risk in the 'Install gh-graph' and 'Recommend + implement + open PR' steps.

