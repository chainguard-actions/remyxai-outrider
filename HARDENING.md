<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.6.11** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are directly interpolated inside run: shell command strings in action.yml. In the 'Install gh-graph selection-pass tool on PATH' step, `${{ github.action_path }}` is embedded directly in the shell command: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. In the 'Recommend + implement + open PR' step, `${{ github.action_path }}` is embedded directly in: `python ${{ github.action_path }}/src/run.py`. Any ${{ }} expression in a run: block is a script-injection risk because YAML template substitution happens before the shell ever sees the value.

Locations:

- `action.yml:175`
- `action.yml:202`

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are directly interpolated inside a run: shell command string in the workflow file. In the 'Mint Remyx bot token' step, both `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` are embedded directly in the curl command string inside the run: block. `github.repository` in particular can contain shell metacharacters (e.g. a repo named with special chars). These values should be passed via env: variables and referenced as shell variables (e.g. $REMYX_API_KEY, $GITHUB_REPOSITORY) instead.

Locations:

- `.github/workflows/outrider.yml:31`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the tag is moved. Failing references in action.yml: `uses: actions/setup-python@v5` and `uses: actions/setup-node@v4`. Failing reference in .github/workflows/outrider.yml: `uses: actions/checkout@v4`. All should be pinned to full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:163`
- `action.yml:168`
- `.github/workflows/outrider.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all three findings: (1) Script injection in action.yml 'Install gh-graph' step: moved `${{ github.action_path }}` into env var ACTION_PATH. (2) Script injection in action.yml 'Recommend + implement + open PR' step: added ACTION_PATH to the existing env block and replaced the inline expression with $ACTION_PATH. (3) Script injection in outrider.yml 'Mint Remyx bot token' step: moved `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` into step-level env vars (REMYX_API_KEY, GH_REPOSITORY) and referenced them as shell variables. (4) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065. (5) Pinned actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020. (6) Pinned actions/checkout@v4 to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5.

