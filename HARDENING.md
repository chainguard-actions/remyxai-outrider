<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.6.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two run: blocks in action.yml directly interpolate ${{ github.action_path }} — a github.* context expression — into shell command strings. Per the script-injection check, any ${{ ... }} expression interpolated directly inside a run: block is a violation (sub-rule a), regardless of which context it reads from. (1) The 'Install gh-graph selection-pass tool on PATH' step uses: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) The 'Recommend + implement + open PR' step uses: `python ${{ github.action_path }}/src/run.py`.

Locations:

- `action.yml:183`
- `action.yml:230`

### script-injection (severity: high)

The 'Mint Remyx bot token' run: block in outrider.yml directly interpolates ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} into the shell command string (sub-rule a). The offending line: `token="$(curl -sf -X POST -H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}" -H "Content-Type: application/json" -d "{\"repo\": \"${{ github.repository }}\"}" "https://engine.remyx.ai/api/v1.0/github/installation-token" | python3 -c "import sys, json; print(json.load(sys.stdin).get('token', ''))")" || token=""`. These expressions are expanded by the Actions template engine before the shell ever sees the command, enabling injection of shell metacharacters.

Locations:

- `.github/workflows/outrider.yml:32`

### unpinned-uses (severity: high)

action.yml references two Actions by mutable version tags instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved: (1) `uses: actions/setup-python@v5` (2) `uses: actions/setup-node@v4`.

Locations:

- `action.yml:163`
- `action.yml:168`

### unpinned-uses (severity: high)

The workflow file outrider.yml references an Action by a mutable version tag instead of a full 40-character commit SHA: `uses: actions/checkout@v4`. If this tag is moved or compromised, the workflow will silently execute different code.

Locations:

- `.github/workflows/outrider.yml:23`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all four findings across two files:

**action.yml:**
- Pinned `actions/setup-python@v5` → `@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`
- Pinned `actions/setup-node@v4` → `@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`
- Fixed script injection in 'Install gh-graph' step: moved `${{ github.action_path }}` to `env: ACTION_PATH: ${{ github.action_path }}` and referenced as `$ACTION_PATH` in the run block
- Fixed script injection in 'Recommend + implement + open PR' step: added `ACTION_PATH: ${{ github.action_path }}` to the existing env block and replaced `python ${{ github.action_path }}/src/run.py` with `python "$ACTION_PATH/src/run.py"`

**.github/workflows/outrider.yml:**
- Pinned `actions/checkout@v4` → `@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4`
- Fixed script injection in 'Mint Remyx bot token' step: moved `${{ secrets.REMYX_API_KEY }}` to `env: MINT_API_KEY` and `${{ github.repository }}` to `env: GH_REPOSITORY`, then referenced as plain env vars in the shell script

