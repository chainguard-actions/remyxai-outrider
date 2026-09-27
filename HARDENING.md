<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.6.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings (sub-rule a). Any `${{ ... }}` expression inside a `run:` script is a script-injection risk because YAML template substitution occurs before the shell ever sees the value. (1) The 'Install gh-graph selection-pass tool on PATH' step uses: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) The 'Recommend + implement + open PR' step uses: `python ${{ github.action_path }}/src/run.py`. These should be replaced with the equivalent environment variable `$GITHUB_ACTION_PATH` which is already set by the runner and does not require expression interpolation.

Locations:

- `action.yml:181`
- `action.yml:218`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`. These should be replaced with their full SHA digests, e.g. `actions/setup-python@<40-char-sha> # v5`.

Locations:

- `action.yml:163`
- `action.yml:167`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all four issues in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to commit SHA a26af69be951a213d495a4c3e4e4022e16d87065. (2) Pinned actions/setup-node@v4 to commit SHA 49933ea5288caeca8642d1e84afbd3f7d6820020. (3) Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` in the 'Install gh-graph selection-pass tool on PATH' step (line 181). (4) Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` in the 'Recommend + implement + open PR' step (line 218), also adding quotes around the path for robustness.

