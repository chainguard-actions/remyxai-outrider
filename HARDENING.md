<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.5.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references two composite action steps using mutable tag refs instead of pinned 40-character SHA digests. `actions/setup-python@v5` and `actions/setup-node@v4` can be silently updated by the upstream maintainer, enabling supply-chain attacks.

Locations:

- `action.yml:196`
- `action.yml:201`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. The final step's run block contains `python ${{ github.action_path }}/src/run.py`, which injects the expression value through YAML template substitution before the shell processes it. Any `${{ ... }}` directly in a `run:` block is a script-injection risk. Fix: use the `$GITHUB_ACTION_PATH` environment variable instead (e.g. `python "$GITHUB_ACTION_PATH/src/run.py"`).

Locations:

- `action.yml:248`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings in action.yml: (1) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065 with # v5 comment. (2) Pinned actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020 with # v4 comment. (3) Replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"` to eliminate script injection — $GITHUB_ACTION_PATH is a built-in GitHub Actions environment variable that is always set and avoids YAML template substitution before shell processing.

