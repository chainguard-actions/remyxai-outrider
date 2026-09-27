<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.6.10** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection risk because the expression is substituted by the Actions template engine before the shell ever sees the string, bypassing shell quoting. (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` — the Install gh-graph step. (2) `python ${{ github.action_path }}/src/run.py` — the Recommend + implement + open PR step. Both should use the `$GITHUB_ACTION_PATH` environment variable instead (e.g. `python "$GITHUB_ACTION_PATH/src/run.py"`).

Locations:

- `action.yml:207`
- `action.yml:248`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or the repository is compromised. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`. Each should be replaced with the full SHA digest of the intended release, e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `action.yml:189`
- `action.yml:194`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all four issues in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065. (2) Pinned actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020. (3) Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` in the Install gh-graph step (line 207). (4) Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` in the Recommend + implement + open PR step (line 248), also adding quotes around the path for robustness.

