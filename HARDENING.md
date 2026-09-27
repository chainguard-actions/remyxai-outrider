<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.6.11** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because YAML template substitution occurs before the shell ever sees the string. (1) In the 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) In the 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py`. Both should use the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:221`
- `action.yml:260`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`. Both should be pinned to their full commit SHA (e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`).

Locations:

- `action.yml:200`
- `action.yml:204`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all findings in hardened/action/action.yml:
1. Pinned `actions/setup-python@v5` to SHA `a26af69be951a213d495a4c3e4e4022e16d87065` (# v5)
2. Pinned `actions/setup-node@v4` to SHA `49933ea5288caeca8642d1e84afbd3f7d6820020` (# v4)
3. Replaced `${{ github.action_path }}/src/gh_graph.py` with `$GITHUB_ACTION_PATH/src/gh_graph.py` in the 'Install gh-graph selection-pass tool on PATH' step
4. Replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"` in the 'Recommend + implement + open PR' step

