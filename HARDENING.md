<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.11** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or hijacked.

In action.yml:
- `uses: actions/setup-python@v5` (tag ref)
- `uses: actions/setup-node@v4` (tag ref)

In .github/workflows/outrider.yml:
- `uses: actions/checkout@v4` (tag ref)

All should be pinned to their full SHA digest, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:282`
- `action.yml:286`
- `.github/workflows/outrider.yml:20`

### script-injection (severity: high)

GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings, which causes the expression value to be substituted into the shell script before the shell parses it. This is a script-injection risk.

(a) In action.yml — `Install gh-graph selection-pass tool on PATH` step:
```
install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph
```
`${{ github.action_path }}` is interpolated directly into the shell command.

(a) In action.yml — `Recommend + implement + open PR` step:
```
python ${{ github.action_path }}/src/run.py
```
`${{ github.action_path }}` is interpolated directly into the shell command.

(a) In .github/workflows/outrider.yml — `Mint Remyx bot token` step:
```
token="$(curl -sf -X POST \
  -H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}" \
  -d "{\"repo\": \"${{ github.repository }}\"}" ...)"
```
Both `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` are interpolated directly into the shell command string. These should be passed via `env:` variables and referenced as `$ENV_VAR` in the shell script.

Locations:

- `action.yml:298`
- `action.yml:370`
- `.github/workflows/outrider.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all findings:

1. unpinned-uses: Pinned three mutable tag references to full commit SHAs:
   - actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5 (action.yml)
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4 (action.yml)
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4 (.github/workflows/outrider.yml)

2. script-injection: Moved all ${{ }} expressions from run: shell strings into env: blocks:
   - action.yml 'Install gh-graph' step: ACTION_PATH env var replaces ${{ github.action_path }}
   - action.yml 'Recommend + implement + open PR' step: ACTION_PATH env var added to existing env block, run command updated
   - outrider.yml 'Mint Remyx bot token' step: REMYX_API_KEY and GH_REPOSITORY env vars replace ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} in the curl command

