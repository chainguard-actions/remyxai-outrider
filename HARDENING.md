<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.10** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or hijacked.

In action.yml:
- `uses: actions/setup-python@v5` (tag ref)
- `uses: actions/setup-node@v4` (tag ref)

In .github/workflows/outrider.yml:
- `uses: actions/checkout@v4` (tag ref)

All should be pinned to full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:265`
- `action.yml:270`
- `.github/workflows/outrider.yml:23`

### script-injection (severity: high)

GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings. Before the shell ever sees the string, GitHub's template engine substitutes the value verbatim — if the value contains shell metacharacters (`;`, `|`, `$(...)`, backticks, etc.) they will be interpreted by the shell, enabling command injection.

**Finding 1 — .github/workflows/outrider.yml, "Mint Remyx bot token" step (rule a):**
The run: block embeds `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` directly in the shell command:
```
token="$(curl -sf -X POST \
  -H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}" \
  -d "{\"repo\": \"${{ github.repository }}\"}" ...
```
`github.repository` is attacker-influenced (fork names, renamed repos) and must not be interpolated directly. Both values should be passed via `env:` variables and referenced as `$ENV_VAR` (double-quoted) in the shell.

**Finding 2 — action.yml, "Install gh-graph selection-pass tool on PATH" step (rule a):**
`${{ github.action_path }}` is interpolated directly in the run: block:
```
install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph
```
Should use the `$GITHUB_ACTION_PATH` environment variable instead.

**Finding 3 — action.yml, "Recommend + implement + open PR" step (rule a):**
`${{ github.action_path }}` is interpolated directly in the run: block:
```
python ${{ github.action_path }}/src/run.py
```
Should use `$GITHUB_ACTION_PATH` instead.

Locations:

- `.github/workflows/outrider.yml:33`
- `action.yml:284`
- `action.yml:340`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all findings:

1. unpinned-uses: Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065, actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, and actions/checkout@v4 to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5. All retain the original tag in a comment for readability.

2. script-injection: (a) In action.yml 'Install gh-graph' step: replaced ${{ github.action_path }} with $GITHUB_ACTION_PATH (built-in env var, no injection risk). (b) In action.yml 'Recommend + implement + open PR' step: same replacement. (c) In outrider.yml 'Mint Remyx bot token' step: moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} into the step's env: block as REMYX_API_KEY and GH_REPOSITORY respectively, then referenced them as plain shell variables in the run: block.

