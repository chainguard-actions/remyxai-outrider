<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.6.5** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Install gh-graph selection-pass tool on PATH' step in action.yml directly interpolates `${{ github.action_path }}` inside a run: shell command string: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. Any ${{ }} expression inside a run: block is a script-injection risk.

Locations:

- `action.yml:168`

### script-injection (severity: high)

Sub-rule (a): The 'Recommend + implement + open PR' step in action.yml directly interpolates `${{ github.action_path }}` inside a run: shell command string: `python ${{ github.action_path }}/src/run.py`. Any ${{ }} expression inside a run: block is a script-injection risk.

Locations:

- `action.yml:196`

### script-injection (severity: high)

Sub-rule (a): The 'Mint Remyx bot token' step in outrider.yml directly interpolates `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` inside a run: shell command string (inside the curl invocation). Any ${{ }} expression inside a run: block is a script-injection risk.

Locations:

- `.github/workflows/outrider.yml:30`

### unpinned-uses (severity: high)

action.yml references two actions using mutable version tags instead of full 40-character commit SHAs: `actions/setup-python@v5` and `actions/setup-node@v4`. These are vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `action.yml:155`
- `action.yml:161`

### unpinned-uses (severity: high)

.github/workflows/outrider.yml references `actions/checkout@v4` using a mutable version tag instead of a full 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `.github/workflows/outrider.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 5 findings across action.yml and .github/workflows/outrider.yml:

1. action.yml - Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5
2. action.yml - Pinned actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4
3. action.yml (line 168) - Fixed script injection in 'Install gh-graph selection-pass tool on PATH' step: moved ${{ github.action_path }} into ACTION_PATH env var, replaced inline expression with $ACTION_PATH in run command
4. action.yml (line 196) - Fixed script injection in 'Recommend + implement + open PR' step: added ACTION_PATH: ${{ github.action_path }} to the existing env block, replaced inline expression with $ACTION_PATH in run command
5. outrider.yml - Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4
6. outrider.yml (line 30) - Fixed script injection in 'Mint Remyx bot token' step: moved ${{ secrets.REMYX_API_KEY }} into REMYX_API_KEY env var and ${{ github.repository }} into GH_REPOSITORY env var; replaced inline expressions with plain env var references in the curl command

