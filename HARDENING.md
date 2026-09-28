<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.5.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references two actions using mutable version tags instead of pinned 40-character SHA digests. `actions/setup-python@v5` and `actions/setup-node@v4` can be silently updated by the upstream maintainer, enabling supply-chain attacks. Both must be pinned to a full commit SHA (e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`).

Locations:

- `action.yml:143`
- `action.yml:148`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string. The step 'Recommend + implement + open PR' contains: `python ${{ github.action_path }}/src/run.py`. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection risk because the value is substituted into the shell command string before the shell parses it. The safe alternative is to use the `$GITHUB_ACTION_PATH` environment variable, which is already available in composite action steps: `python "$GITHUB_ACTION_PATH/src/run.py"`.

Locations:

- `action.yml:175`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three issues in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065 # v5; (2) Pinned actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020 # v4; (3) Replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"` to eliminate the script-injection risk — $GITHUB_ACTION_PATH is the built-in environment variable already available in composite action steps and does not require expression interpolation.

