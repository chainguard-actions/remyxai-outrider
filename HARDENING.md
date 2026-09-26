<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.20** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two run: blocks in action.yml directly interpolate ${{ github.action_path }} inside shell command strings, violating rule (a) — any ${{ ... }} expression inside a run: block is a script-injection risk. (1) In the 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) In the 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py`. These should be replaced with the equivalent env-var form (e.g. `"$GITHUB_ACTION_PATH"`) which is already set by the runner without template substitution.

Locations:

- `action.yml:311`
- `action.yml:436`

### unpinned-uses (severity: high)

Two uses: references in action.yml pin to mutable version tags instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or the repository is compromised. Failing references: `uses: actions/setup-python@v5` and `uses: actions/setup-node@v4`. Each should be replaced with the full commit SHA, e.g. `uses: actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `action.yml:291`
- `action.yml:296`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to full SHA a26af69be951a213d495a4c3e4e4022e16d87065 and actions/setup-node@v4 to full SHA 49933ea5288caeca8642d1e84afbd3f7d6820020. (2) Replaced both ${{ github.action_path }} template expressions with the equivalent $GITHUB_ACTION_PATH environment variable — one in the 'Install gh-graph selection-pass tool on PATH' step and one in the 'Recommend + implement + open PR' step. GITHUB_ACTION_PATH is set by the runner and does not require template substitution, eliminating the script-injection risk.

