<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.14

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.14** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two run: blocks in action.yml directly interpolate ${{ github.action_path }} inside shell command strings. Any ${{ ... }} expression inside a run: block is a script-injection risk because the expression is substituted by the template engine before the shell ever sees the string, allowing injection of shell metacharacters. (1) In the 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) In the 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py`. Both should use the $GITHUB_ACTION_PATH environment variable instead.

Locations:

- `action.yml:260`
- `action.yml:370`

### unpinned-uses (severity: high)

Two uses: references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs. This exposes the action to supply-chain attacks if the upstream action's tag is moved or the repository is compromised. Failing references: (1) uses: actions/setup-python@v5 — should be pinned to a full SHA, e.g. actions/setup-python@<40-char-sha> # v5. (2) uses: actions/setup-node@v4 — should be pinned to a full SHA, e.g. actions/setup-node@<40-char-sha> # v4.

Locations:

- `action.yml:234`
- `action.yml:239`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to @a26af69be951a213d495a4c3e4e4022e16d87065 # v5. (2) Pinned actions/setup-node@v4 to @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4. (3) Replaced both ${{ github.action_path }} template expressions in run: blocks with the $GITHUB_ACTION_PATH environment variable — one in the 'Install gh-graph selection-pass tool on PATH' step and one in the 'Recommend + implement + open PR' step. The python invocation was also quoted for robustness.

