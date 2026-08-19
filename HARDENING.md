<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.5.2** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml pin to mutable version tags instead of immutable 40-character commit SHAs. If the upstream action is compromised or the tag is moved, the action will silently execute different code. Failing references:
- `uses: actions/setup-python@v5` (line 149)
- `uses: actions/setup-node@v4` (line 153)
These should be pinned to their full SHA digests, e.g. `actions/setup-python@<40-char-sha> # v5`.

Locations:

- `action.yml:149`
- `action.yml:153`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. The offending line is:

  `python ${{ github.action_path }}/src/run.py`

Any `${{ ... }}` expression inside a `run:` block undergoes YAML template substitution before the shell ever sees the value, making it a script-injection risk. The safe alternative is to use the pre-set environment variable `$GITHUB_ACTION_PATH` instead, which is already available in composite action steps without any interpolation.

Locations:

- `action.yml:185`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. Pinned `actions/setup-python@v5` → `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5` (line 149)
2. Pinned `actions/setup-node@v4` → `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4` (line 153)
3. Replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"` to eliminate script injection — `$GITHUB_ACTION_PATH` is a pre-set environment variable available in composite action steps and does not undergo YAML template substitution.

### Iteration 2

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned actions/checkout@v4 to full SHA 11d5960a326750d5838078e36cf38b85af677262 (tag preserved as comment). 2. Moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} out of the run: shell string into a step-level env: block (as REMYX_API_KEY and REPO respectively), then referenced them as plain shell variables in the curl command to eliminate script injection risk.

