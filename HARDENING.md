<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.6.8** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two run: blocks in action.yml directly interpolate ${{ github.action_path }} into shell command strings. In the 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. In the 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py`. Any ${{ }} expression interpolated directly inside a run: shell command is a script-injection risk — the value flows through YAML template substitution before the shell ever sees it, bypassing shell quoting.

Locations:

- `action.yml:160`
- `action.yml:196`

### script-injection (severity: high)

Sub-rule (a): The 'Mint Remyx bot token' run: block in outrider.yml directly interpolates ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} into the shell command string inside a curl invocation: `curl -sf -X POST -H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}" -d "{\"repo\": \"${{ github.repository }}\"}" ...`. These ${{ }} expressions are substituted by the Actions template engine before the shell parses the command, meaning special characters in the values (e.g. from github.repository) could alter the shell command structure.

Locations:

- `.github/workflows/outrider.yml:32`

### unpinned-uses (severity: high)

action.yml references two actions by mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved: `uses: actions/setup-python@v5` and `uses: actions/setup-node@v4`. These should be pinned to full SHA digests (e.g. actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5).

Locations:

- `action.yml:160`
- `action.yml:165`

### unpinned-uses (severity: high)

outrider.yml references actions/checkout by the mutable tag @v4 instead of an immutable 40-character commit SHA: `uses: actions/checkout@v4`. This should be pinned to a full SHA digest (e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4).

Locations:

- `.github/workflows/outrider.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all four findings:
1. action.yml script-injection (line 160): Moved `${{ github.action_path }}` to env var `ACTION_PATH` in the 'Install gh-graph' step.
2. action.yml script-injection (line 196): Added `ACTION_PATH: ${{ github.action_path }}` to the existing env block of the 'Recommend + implement + open PR' step; updated run command to use `"$ACTION_PATH/src/run.py"`.
3. outrider.yml script-injection (line 32): Moved `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` into a step-level env block as `REMYX_API_KEY` and `GH_REPOSITORY`; updated curl command to reference plain env vars.
4. action.yml unpinned-uses: Pinned `actions/setup-python@v5` → `@a26af69be951a213d495a4c3e4e4022e16d87065 # v5` and `actions/setup-node@v4` → `@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`.
5. outrider.yml unpinned-uses: Pinned `actions/checkout@v4` → `@11d5960a326750d5838078e36cf38b85af677262 # v4`.

