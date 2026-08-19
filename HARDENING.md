<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.6.10** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.action_path }}` is interpolated directly inside a `run:` shell command string in the 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. Any `${{ ... }}` expression directly in a `run:` block is a script-injection risk regardless of the context it reads from.

Locations:

- `action.yml:196`

### script-injection (severity: high)

Sub-rule (a): `${{ github.action_path }}` is interpolated directly inside a `run:` shell command string in the 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py`. Any `${{ ... }}` expression directly in a `run:` block is a script-injection risk.

Locations:

- `action.yml:243`

### script-injection (severity: high)

Sub-rule (a): The 'Mint Remyx bot token' step in the workflow interpolates `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` directly inside a `run:` shell command string. Specifically: `-H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}"` and `-d "{\"repo\": \"${{ github.repository }}\"}"`. All `${{ ... }}` expressions in `run:` blocks are script-injection findings.

Locations:

- `.github/workflows/outrider.yml:30`

### unpinned-uses (severity: high)

action.yml references two actions by mutable version tags instead of full 40-character commit SHAs: `actions/setup-python@v5` and `actions/setup-node@v4`. These are vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `action.yml:168`
- `action.yml:173`

### unpinned-uses (severity: high)

The workflow file references `actions/checkout@v4` by a mutable version tag instead of a full 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `.github/workflows/outrider.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 5 findings:
1. action.yml: Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5
2. action.yml: Pinned actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4
3. .github/workflows/outrider.yml: Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4
4. action.yml line 196: Moved ${{ github.action_path }} into ACTION_PATH env var in the 'Install gh-graph' step; run block now uses $ACTION_PATH
5. action.yml line 243: Added ACTION_PATH: ${{ github.action_path }} to the existing env block of the 'Recommend + implement + open PR' step; run block now uses python "$ACTION_PATH/src/run.py"
6. .github/workflows/outrider.yml line 30: Moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} into REMYX_API_KEY and REPOSITORY env vars in the 'Mint Remyx bot token' step; curl command now references $REMYX_API_KEY and $REPOSITORY

