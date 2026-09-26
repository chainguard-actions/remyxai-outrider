<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.13** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings (sub-rule a). Although `github.action_path` is GitHub-controlled, any `${{ ... }}` expression directly inside a `run:` script is a script-injection finding per the check rules. (1) The 'Install gh-graph selection-pass tool on PATH' step runs: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) The 'Recommend + implement + open PR' step runs: `python ${{ github.action_path }}/src/run.py`. Both should use the `$GITHUB_ACTION_PATH` environment variable instead of the `${{ ... }}` expression interpolation.

Locations:

- `action.yml:330`
- `action.yml:460`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved or hijacked. In action.yml: `actions/setup-python@v5` and `actions/setup-node@v4`. In examples/workflows/with-cocoindex.yml: `actions/checkout@v4` and `remyxai/outrider@v1`. All should be pinned to full SHA digests (e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`).

Locations:

- `action.yml:308`
- `action.yml:313`
- `examples/workflows/with-cocoindex.yml:33`
- `examples/workflows/with-cocoindex.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed two script-injection findings in action.yml by replacing ${{ github.action_path }} with $GITHUB_ACTION_PATH in both run: blocks (lines 330 and 460). Fixed four unpinned-uses findings: pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065, actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020 in action.yml; pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 and remyxai/outrider@v1 to SHA 1dcbff5c76ec301b1f1dd2bbaa7071fb6fe07b62 in examples/workflows/with-cocoindex.yml. Also repaired a file corruption that occurred during editing where the REMYX_RECOMMENDATION_LIMIT env var line had been merged with the run: block.

