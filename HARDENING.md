<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.6.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two composite action steps use mutable tag-based refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tags are moved or compromised. Failing references: `uses: actions/setup-python@v5` and `uses: actions/setup-node@v4`.

Locations:

- `action.yml:179`
- `action.yml:184`

### script-injection (severity: high)

Two `run:` blocks directly interpolate `${{ github.action_path }}` (a `github.*` context expression) inside shell command strings (sub-rule a). Although `github.action_path` is not attacker-controlled in the same way as `github.head_ref`, any `${{ ... }}` expression interpolated directly into a `run:` shell string is a script-injection finding per the check rules. Offending lines: (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the 'Install gh-graph selection-pass tool on PATH' step; (2) `python ${{ github.action_path }}/src/run.py` in the 'Recommend + implement + open PR' step. Fix: use the `$GITHUB_ACTION_PATH` environment variable instead of the expression.

Locations:

- `action.yml:200`
- `action.yml:248`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all four issues in action.yml: (1) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065; (2) Pinned actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020; (3) Replaced ${{ github.action_path }} with $GITHUB_ACTION_PATH in the 'Install gh-graph selection-pass tool on PATH' step; (4) Replaced ${{ github.action_path }} with $GITHUB_ACTION_PATH (also added quotes) in the 'Recommend + implement + open PR' step. Original tags preserved as comments for readability.

