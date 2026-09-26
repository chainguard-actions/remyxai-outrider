<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.10** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings (sub-rule a: any `${{ ... }}` expression in a `run:` block is a script-injection risk). (1) The "Install gh-graph selection-pass tool on PATH" step runs: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) The "Recommend + implement + open PR" step runs: `python ${{ github.action_path }}/src/run.py`. Both should use the `$GITHUB_ACTION_PATH` environment variable instead of the `${{ github.action_path }}` expression.

Locations:

- `action.yml:15083`
- `action.yml:21175`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`. Each should be replaced with the full commit SHA, e.g. `actions/setup-python@<40-hex-sha> # v5`.

Locations:

- `action.yml:14250`
- `action.yml:14376`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. Pinned `actions/setup-python@v5` → `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`
2. Pinned `actions/setup-node@v4` → `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`
3. Replaced both `${{ github.action_path }}` expressions in `run:` blocks with `$GITHUB_ACTION_PATH` (the equivalent pre-set environment variable), eliminating the script-injection risk in the 'Install gh-graph selection-pass tool on PATH' step and the 'Recommend + implement + open PR' step.

