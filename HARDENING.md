<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.6.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml pin to mutable version tags instead of full 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised.

Failing references:
- `uses: actions/setup-python@v5` (line ~163)
- `uses: actions/setup-node@v4` (line ~168)

These should be pinned to their full SHA digests, e.g. `uses: actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `action.yml:163`
- `action.yml:168`

### script-injection (severity: high)

Two `run:` blocks directly interpolate `${{ github.action_path }}` — a `github.*` context expression — inside shell command strings (sub-rule a). Any `${{ ... }}` expression interpolated directly into a `run:` script is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it, bypassing shell quoting.

Offending lines:
1. `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the "Install gh-graph selection-pass tool on PATH" step.
2. `python ${{ github.action_path }}/src/run.py` in the "Recommend + implement + open PR" step.

Fix: use the `$GITHUB_ACTION_PATH` environment variable (already set by the runner) instead of the `${{ github.action_path }}` expression, e.g. `python "$GITHUB_ACTION_PATH/src/run.py"`.

Locations:

- `action.yml:183`
- `action.yml:228`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all four issues in actions/hardened/remyxai--outrider/v1.6.0/action.yml:
1. Pinned `actions/setup-python@v5` → `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`
2. Pinned `actions/setup-node@v4` → `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`
3. Replaced `"${{ github.action_path }}/src/gh_graph.py"` with `"$GITHUB_ACTION_PATH/src/gh_graph.py"` in the gh-graph install step
4. Replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"` in the recommend step (also added proper double-quoting)

