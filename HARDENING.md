<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.16

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.16** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised:
- `uses: actions/setup-python@v5` (mutable tag @v5)
- `uses: actions/setup-node@v4` (mutable tag @v4)
These should be pinned to their full commit SHAs, e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `action.yml:304`
- `action.yml:309`

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings (sub-rule a). Any `${{ ... }}` expression interpolated directly into a `run:` script is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it, bypassing shell quoting.

1. "Install gh-graph selection-pass tool on PATH" step:
   `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`

2. "Recommend + implement + open PR" step:
   `python ${{ github.action_path }}/src/run.py`

Fix: use the `$GITHUB_ACTION_PATH` environment variable (already set by the runner) instead of the `${{ github.action_path }}` expression, e.g.:
  `install -m 0755 "$GITHUB_ACTION_PATH/src/gh_graph.py" /usr/local/bin/gh-graph`
  `python "$GITHUB_ACTION_PATH/src/run.py"`

Locations:

- `action.yml:318`
- `action.yml:393`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all four issues in hardened/action/action.yml:
1. Pinned `actions/setup-python@v5` to `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`
2. Pinned `actions/setup-node@v4` to `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`
3. Replaced `"${{ github.action_path }}/src/gh_graph.py"` with `"$GITHUB_ACTION_PATH/src/gh_graph.py"` in the gh-graph install step
4. Replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"` in the recommend step (also added quotes around the path)

