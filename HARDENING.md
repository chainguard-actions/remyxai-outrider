<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.6.5** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings (rule a). Any `${{ ... }}` expression interpolated directly into a `run:` shell script is a script-injection risk, as the value is substituted by the Actions template engine before the shell ever sees it.

1. "Install gh-graph selection-pass tool on PATH" step:
   `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`

2. "Recommend + implement + open PR" step:
   `python ${{ github.action_path }}/src/run.py`

Fix: use the `$GITHUB_ACTION_PATH` environment variable (already set by the runner) instead of the `${{ github.action_path }}` expression, e.g. `python "$GITHUB_ACTION_PATH/src/run.py"`

Locations:

- `action.yml:183`
- `action.yml:222`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml pin to mutable version tags rather than immutable 40-character commit SHAs. This exposes the action to supply-chain attacks if the upstream action is compromised or the tag is moved.

- `uses: actions/setup-python@v5` (should be pinned to a full SHA, e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`)
- `uses: actions/setup-node@v4` (should be pinned to a full SHA, e.g. `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`)

Locations:

- `action.yml:164`
- `action.yml:169`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all findings in action.yml:
1. unpinned-uses: Pinned `actions/setup-python@v5` to full SHA `a26af69be951a213d495a4c3e4e4022e16d87065` and `actions/setup-node@v4` to full SHA `49933ea5288caeca8642d1e84afbd3f7d6820020`, preserving the version tag as a comment.
2. script-injection: Replaced both occurrences of `${{ github.action_path }}` in `run:` blocks with the `$GITHUB_ACTION_PATH` environment variable (already set by the runner), which avoids template-engine interpolation before the shell sees the script. Also added double-quotes around the path in the `python` invocation for robustness.

