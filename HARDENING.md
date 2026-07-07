<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.7.10** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags instead of immutable 40-character commit SHAs. In action.yml: `actions/setup-python@v5` and `actions/setup-node@v4`. In .github/workflows/outrider.yml: `actions/checkout@v4`. Any of these tags could be moved to point to a different (potentially malicious) commit without changing the workflow.

Locations:

- `action.yml:258`
- `action.yml:263`
- `.github/workflows/outrider.yml:23`

### script-injection (severity: high)

Rule (a): `${{ ... }}` expressions are directly interpolated inside `run:` shell command strings. In action.yml, the 'Install gh-graph selection-pass tool on PATH' step uses `${{ github.action_path }}` directly in the shell command: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. Similarly, the final 'Recommend + implement + open PR' step uses `python ${{ github.action_path }}/src/run.py`. In .github/workflows/outrider.yml, the 'Mint Remyx bot token' step directly interpolates `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` inside the shell `run:` block. Any `${{ ... }}` expression inside a `run:` block is substituted before the shell parses the command, enabling injection if the value contains shell metacharacters.

Locations:

- `action.yml:271`
- `action.yml:358`
- `.github/workflows/outrider.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three unpinned uses references: actions/setup-python@v5 → SHA a26af69..., actions/setup-node@v4 → SHA 49933ea..., actions/checkout@v4 → SHA 34e1148... Fixed script injection in action.yml by moving ${{ github.action_path }} into env: blocks (as ACTION_PATH) for both the 'Install gh-graph' step and the 'Recommend + implement + open PR' step. Fixed script injection in .github/workflows/outrider.yml by moving ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} into a step-level env: block for the 'Mint Remyx bot token' step. The examples/workflows/with-cocoindex.yml file was left unchanged as it is a user-facing template/example file, not part of the action's own CI infrastructure.

