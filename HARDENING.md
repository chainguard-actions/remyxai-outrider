<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.6.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two composite action steps use mutable tag references instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tags are moved or hijacked:
- `uses: actions/setup-python@v5` (should be pinned to a full SHA)
- `uses: actions/setup-node@v4` (should be pinned to a full SHA)

Locations:

- `action.yml:167`
- `action.yml:172`

### script-injection (severity: high)

Two `run:` blocks directly interpolate `${{ github.action_path }}` (a `github.*` context expression) into shell command strings. Per rule (a), ANY `${{ ... }}` expression inside a `run:` script is a script-injection finding regardless of which context it reads from, because the value flows through YAML template substitution before the shell ever sees it.

1. Step "Install gh-graph selection-pass tool on PATH":
   `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`

2. Step "Recommend + implement + open PR":
   `python ${{ github.action_path }}/src/run.py`

Fix: use the `$GITHUB_ACTION_PATH` environment variable (already set by the runner) instead of the `${{ github.action_path }}` expression, e.g. `python "$GITHUB_ACTION_PATH/src/run.py"`.

Locations:

- `action.yml:185`
- `action.yml:218`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed four issues in hardened/action/action.yml:
1. Pinned `actions/setup-python@v5` → `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`
2. Pinned `actions/setup-node@v4` → `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`
3. Replaced `"${{ github.action_path }}/src/gh_graph.py"` with `"$GITHUB_ACTION_PATH/src/gh_graph.py"` in the gh-graph install step
4. Replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"` in the recommend step (also added proper quoting)

