<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.6.11** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings (rule a: any `${{ ... }}` expression inside a `run:` block is a script-injection risk). (1) The 'Install gh-graph selection-pass tool on PATH' step uses: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) The 'Recommend + implement + open PR' step uses: `python ${{ github.action_path }}/src/run.py`. These should be replaced with the pre-set `$GITHUB_ACTION_PATH` environment variable instead of the `${{ ... }}` expression form.

Locations:

- `action.yml:196`
- `action.yml:243`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved: (1) `uses: actions/setup-python@v5` — should be pinned to a full SHA digest. (2) `uses: actions/setup-node@v4` — should be pinned to a full SHA digest.

Locations:

- `action.yml:183`
- `action.yml:188`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all four issues in action.yml: (1) Pinned `actions/setup-python@v5` to full SHA `a26af69be951a213d495a4c3e4e4022e16d87065` with `# v5` comment. (2) Pinned `actions/setup-node@v4` to full SHA `49933ea5288caeca8642d1e84afbd3f7d6820020` with `# v4` comment. (3) Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` in the 'Install gh-graph' step's `run:` block. (4) Replaced `${{ github.action_path }}` with `$GITHUB_ACTION_PATH` in the 'Recommend + implement + open PR' step's `run:` block (also added quotes around the path for robustness).

