<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.21** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two run: blocks in action.yml directly interpolate ${{ github.action_path }} inside shell command strings (sub-rule a). Any ${{ ... }} expression inside a run: block is a script-injection risk regardless of the context. (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the 'Install gh-graph selection-pass tool on PATH' step. (2) `python ${{ github.action_path }}/src/run.py` in the 'Recommend + implement + open PR' step. These should be replaced with the pre-set $GITHUB_ACTION_PATH environment variable, e.g. `install -m 0755 "$GITHUB_ACTION_PATH/src/gh_graph.py" ...` and `python "$GITHUB_ACTION_PATH/src/run.py"`.

Locations:

- `action.yml:360`
- `action.yml:490`

### unpinned-uses (severity: high)

Two uses: references in action.yml are pinned to mutable version tags rather than immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved. Failing references: `uses: actions/setup-python@v5` and `uses: actions/setup-node@v4`. These should be pinned to their full SHA digests, e.g. `uses: actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `action.yml:336`
- `action.yml:341`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all four issues in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065. (2) Pinned actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020. (3) Replaced ${{ github.action_path }} with $GITHUB_ACTION_PATH in the 'Install gh-graph selection-pass tool on PATH' step. (4) Replaced ${{ github.action_path }} with "$GITHUB_ACTION_PATH" (quoted) in the 'Recommend + implement + open PR' step.

