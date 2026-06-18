<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.6.10** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings (sub-rule a). Any `${{ ... }}` expression inside a `run:` script is a script-injection risk regardless of the context it reads from. Offending lines:
1. `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` — in the 'Install gh-graph selection-pass tool on PATH' step.
2. `python ${{ github.action_path }}/src/run.py` — in the 'Recommend + implement + open PR' step.
These should be replaced with the safe env-var form, e.g. `"$GITHUB_ACTION_PATH"` (the runner automatically sets this env var).

Locations:

- `action.yml:183`
- `action.yml:222`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable version tags instead of full 40-character SHA commit digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or hijacked:
- `uses: actions/setup-python@v5` (should be pinned to a full SHA, e.g. `actions/setup-python@<40-char-sha> # v5`)
- `uses: actions/setup-node@v4` (should be pinned to a full SHA, e.g. `actions/setup-node@<40-char-sha> # v4`)

Locations:

- `action.yml:166`
- `action.yml:170`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all four issues in action.yml:
1. Pinned `actions/setup-python@v5` → `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`
2. Pinned `actions/setup-node@v4` → `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`
3. Replaced `"${{ github.action_path }}/src/gh_graph.py"` with `"$GITHUB_ACTION_PATH/src/gh_graph.py"` in the 'Install gh-graph' step
4. Replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"` in the 'Recommend + implement + open PR' step

