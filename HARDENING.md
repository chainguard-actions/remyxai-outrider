<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.5.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.5.4** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string. The step 'Recommend + implement + open PR' contains `python ${{ github.action_path }}/src/run.py` in its `run:` block. Any `${{ ... }}` expression inside a `run:` script is a script-injection risk because the value is substituted into the shell command string before the shell parses it. The safe alternative is to use the pre-set environment variable `$GITHUB_ACTION_PATH` instead of `${{ github.action_path }}`.

Offending line: `        python ${{ github.action_path }}/src/run.py`

Locations:

- `action.yml:218`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable tag refs instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if the upstream action tags are moved:
- `uses: actions/setup-python@v5` (tag `v5`)
- `uses: actions/setup-node@v4` (tag `v4`)

These should be pinned to their full SHA digests, e.g. `uses: actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `action.yml:160`
- `action.yml:164`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Three changes made to hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to full SHA a26af69be951a213d495a4c3e4e4022e16d87065 with # v5 comment. (2) Pinned actions/setup-node@v4 to full SHA 49933ea5288caeca8642d1e84afbd3f7d6820020 with # v4 comment. (3) Replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"` to eliminate script injection by using the pre-set $GITHUB_ACTION_PATH environment variable instead of a ${{ }} expression interpolated directly into the shell command string.

### Iteration 2

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned actions/checkout@v4 to full SHA 11d5960a326750d5838078e36cf38b85af677262 (# v4 comment preserved). 2. Fixed script injection in 'Mint Remyx bot token' step: moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} into the step's env: block as REMYX_API_KEY and GITHUB_REPOSITORY_NAME respectively, then referenced them as quoted shell variables in the run: script. The job-level REMYX_API_KEY env var was already present but the step-level env: block overrides it safely for this step.

