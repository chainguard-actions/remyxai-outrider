<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.5.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.5.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command string. The step 'Recommend + implement + open PR' uses `python ${{ github.action_path }}/src/run.py` — the `${{ github.action_path }}` expression is substituted into the shell command before the shell ever sees it. Per the check rules, any ${{ ... }} directly inside a run: block is a script-injection finding regardless of which context it reads from.

Locations:

- `action.yml:183`

### unpinned-uses (severity: high)

Two composite action steps reference external actions by mutable version tags instead of full 40-character SHA digests, making them vulnerable to supply-chain attacks if the tag is moved: (1) `uses: actions/setup-python@v5` — tag `v5` is not a SHA; (2) `uses: actions/setup-node@v4` — tag `v4` is not a SHA. These should be pinned to their full commit SHAs (e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`).

Locations:

- `action.yml:146`
- `action.yml:151`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings in action.yml: (1) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065; (2) Pinned actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020; (3) Moved ${{ github.action_path }} out of the run: shell string into the step's env: block as ACTION_PATH, then referenced it as "$ACTION_PATH/src/run.py" in the shell script to eliminate the script-injection risk.

