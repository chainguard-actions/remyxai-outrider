<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.15** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`.

Locations:

- `action.yml:266`
- `action.yml:271`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. The offending line is: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. Any `${{ ... }}` expression directly inside a `run:` block undergoes YAML template substitution before the shell ever sees it, making it a script-injection risk regardless of which context it reads from.

Locations:

- `action.yml:295`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. The offending line is: `python ${{ github.action_path }}/src/run.py`. Any `${{ ... }}` expression directly inside a `run:` block undergoes YAML template substitution before the shell ever sees it, making it a script-injection risk regardless of which context it reads from.

Locations:

- `action.yml:370`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065 and actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, preserving original tags as comments. (2) Moved ${{ github.action_path }} out of the 'Install gh-graph' step's run: block into a new env: block as ACTION_PATH, referencing it as $ACTION_PATH in the shell. (3) Moved ${{ github.action_path }} out of the 'Recommend + implement + open PR' step's run: block into the existing env: block as ACTION_PATH, referencing it as "$ACTION_PATH/src/run.py" in the shell.

