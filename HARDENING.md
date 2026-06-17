<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.5.5** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. The step 'Recommend + implement + open PR' uses `python ${{ github.action_path }}/src/run.py` — the expression `${{ github.action_path }}` is substituted by the YAML template engine before the shell ever sees the string, making it a script-injection risk. The safe alternative is to use the `$GITHUB_ACTION_PATH` environment variable instead: `python "$GITHUB_ACTION_PATH/src/run.py"`

Locations:

- `action.yml:219`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `uses: actions/setup-python@v5` (tag `v5`)
- `uses: actions/setup-node@v4` (tag `v4`)
These should be pinned to their full SHA digests, e.g. `uses: actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `action.yml:163`
- `action.yml:168`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Three fixes applied to actions/hardened/remyxai--outrider/v1.5.5/action.yml:
1. script-injection (line 219): Replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"` — uses the built-in GitHub Actions environment variable instead of a template expression in the shell string.
2. unpinned-uses (line 163): Pinned `actions/setup-python@v5` → `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.
3. unpinned-uses (line 168): Pinned `actions/setup-node@v4` → `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`.

