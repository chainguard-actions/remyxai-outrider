<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.5.5** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command. The line `python ${{ github.action_path }}/src/run.py` embeds the GitHub Actions expression directly into the shell command string before the shell ever sees it. While github.action_path is not attacker-controlled like github.head_ref, the check prohibits any ${{ ... }} inside a run: block. The safe fix is to use the pre-set environment variable $GITHUB_ACTION_PATH instead: `python "$GITHUB_ACTION_PATH/src/run.py"`.

Locations:

- `action.yml:213`

### unpinned-uses (severity: high)

Two uses: references in action.yml use mutable version tags instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the upstream tag is moved or the repository is compromised. Failing references: `uses: actions/setup-python@v5` and `uses: actions/setup-node@v4`. Both should be pinned to their full commit SHA (e.g., `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`).

Locations:

- `action.yml:159`
- `action.yml:163`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three findings in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to full SHA a26af69be951a213d495a4c3e4e4022e16d87065; (2) Pinned actions/setup-node@v4 to full SHA 49933ea5288caeca8642d1e84afbd3f7d6820020; (3) Replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"` to eliminate the inline ${{ }} expression in the run: block, using the pre-set GITHUB_ACTION_PATH environment variable instead.

