<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.5.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.5.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two composite action steps in action.yml reference external actions using mutable version tags instead of full 40-character SHA commit digests. This exposes the action to supply-chain attacks where a tag could be silently moved to point to malicious code.

Failing references:
- `uses: actions/setup-python@v5` (line ~163)
- `uses: actions/setup-node@v4` (line ~168)

These should be pinned to their full SHA, e.g.:
  `uses: actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`
  `uses: actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`

Locations:

- `action.yml:163`
- `action.yml:168`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is directly interpolated inside a `run:` shell command string. In the 'Recommend + implement + open PR' step, the run block contains:

  `python ${{ github.action_path }}/src/run.py`

Any `${{ ... }}` expression inside a `run:` block is subject to YAML template substitution before the shell ever sees the value, making it a script-injection risk. The safe pattern is to use the pre-set `$GITHUB_ACTION_PATH` environment variable instead:

  `python "$GITHUB_ACTION_PATH/src/run.py"`

Locations:

- `action.yml:214`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three issues in actions/hardened/remyxai--outrider/v1.5.4/action.yml: (1) Pinned actions/setup-python@v5 to full SHA a26af69be951a213d495a4c3e4e4022e16d87065 # v5; (2) Pinned actions/setup-node@v4 to full SHA 49933ea5288caeca8642d1e84afbd3f7d6820020 # v4; (3) Replaced ${{ github.action_path }} expression inside the run: block with the pre-set $GITHUB_ACTION_PATH environment variable to eliminate the script-injection risk.

