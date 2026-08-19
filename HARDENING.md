<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references two GitHub Actions using mutable version tags instead of pinned full-length SHA digests. This exposes the action to supply-chain attacks if the upstream tag is moved or the repository is compromised.

Failing references:
- `uses: actions/setup-python@v5` (mutable tag)
- `uses: actions/setup-node@v4` (mutable tag)

Each should be replaced with a full 40-character commit SHA, e.g. `uses: actions/setup-python@a26ac6f7b5b9e2b2b2b2b2b2b2b2b2b2b2b2b2b2 # v5`.

Locations:

- `action.yml:163`
- `action.yml:168`

### unpinned-uses (severity: high)

`.github/workflows/outrider.yml` references a GitHub Action using a mutable version tag instead of a pinned full-length SHA digest.

Failing reference:
- `uses: actions/checkout@v4` (mutable tag)

This should be replaced with a full 40-character commit SHA, e.g. `uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/outrider.yml:20`

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings (sub-rule a). Any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it, bypassing shell quoting.

Offending lines:
1. Step "Install gh-graph selection-pass tool on PATH":
   `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`

2. Step "Recommend + implement + open PR" (also unquoted):
   `python ${{ github.action_path }}/src/run.py`

Fix: use the `$GITHUB_ACTION_PATH` environment variable instead, which is already available in composite action steps and is properly shell-quoted:
   `install -m 0755 "$GITHUB_ACTION_PATH/src/gh_graph.py" /usr/local/bin/gh-graph`
   `python "$GITHUB_ACTION_PATH/src/run.py"`

Locations:

- `action.yml:196`
- `action.yml:233`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings:
1. Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5 in action.yml
2. Pinned actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4 in action.yml
3. Pinned actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5 # v4 in .github/workflows/outrider.yml
4. Replaced both ${{ github.action_path }} script injections in action.yml run: blocks with $GITHUB_ACTION_PATH (the pre-set environment variable), and properly quoted the python invocation.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Mint Remyx bot token' step of .github/workflows/outrider.yml. Removed direct ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} interpolation from the run: shell string. REMYX_API_KEY was already available via the job-level env block so it's referenced as $REMYX_API_KEY. Added REPO: ${{ github.repository }} to the step's env: block and replaced the inline expression with $REPO in the shell script.

