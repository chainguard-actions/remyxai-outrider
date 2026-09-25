<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.5.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two composite action steps in action.yml reference external actions using mutable version tags instead of pinned 40-character SHA commit digests. This exposes the action to supply-chain attacks where a tag could be silently moved to point to malicious code. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`. These must be replaced with their full SHA digests (e.g. `actions/setup-python@<40-hex-sha> # v5`).

Locations:

- `action.yml:158`
- `action.yml:163`

### script-injection (severity: high)

Sub-rule (a) violation: a `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. The offending line is: `python ${{ github.action_path }}/src/run.py`. Any `${{ ... }}` expression embedded in a `run:` block undergoes YAML template substitution before the shell ever sees it, bypassing shell quoting. The safe alternative is to use the pre-set environment variable `$GITHUB_ACTION_PATH` instead: `python "$GITHUB_ACTION_PATH/src/run.py"`.

Locations:

- `action.yml:215`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three issues in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to full SHA a26af69be951a213d495a4c3e4e4022e16d87065 # v5; (2) Pinned actions/setup-node@v4 to full SHA 49933ea5288caeca8642d1e84afbd3f7d6820020 # v4; (3) Replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"` to eliminate the ${{ }} expression from the run: shell block, using the pre-set GITHUB_ACTION_PATH environment variable instead.

