<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.16

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.16** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved.

In action.yml:
- `uses: actions/setup-python@v5` (tag, not SHA)
- `uses: actions/setup-node@v4` (tag, not SHA)

In .github/workflows/outrider.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

In .github/workflows/outrider-daily.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

All should be pinned to their full 40-hex-character commit SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:275`
- `action.yml:280`
- `.github/workflows/outrider.yml:37`
- `.github/workflows/outrider-daily.yml:30`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell command strings (sub-rule a). GitHub Actions performs template substitution before the shell executes the script, so an attacker-controlled value can inject arbitrary shell commands.

**action.yml — 'Install gh-graph selection-pass tool on PATH' step:**
`install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`
`${{ github.action_path }}` (a `github.*` context) is interpolated directly in the `run:` shell string.

**action.yml — 'Recommend + implement + open PR' step:**
`python ${{ github.action_path }}/src/run.py`
Same issue — `${{ github.action_path }}` interpolated directly in shell.

**outrider.yml — 'Configure provider auth' step:**
`if [ "${{ inputs.provider }}" = "zai" ]; then`
`${{ inputs.provider }}` (a workflow_dispatch input, attacker-controllable) is interpolated directly in the `run:` shell string.

**outrider.yml — 'Mint Remyx bot token' step:**
`-H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}"` and `-d "{\"repo\": \"${{ github.repository }}\"}"` — both `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` are interpolated directly in the `run:` shell string. Any `${{ ... }}` in a `run:` block is a script-injection risk regardless of the context it reads from.

Locations:

- `action.yml:310`
- `action.yml:380`
- `.github/workflows/outrider.yml:50`
- `.github/workflows/outrider.yml:62`

### github-env-injection (severity: high)

The 'Configure provider auth' step in outrider.yml writes values derived from workflow-controlled sources to `$GITHUB_ENV` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A newline-containing value can inject additional environment variable assignments into subsequent steps.

1. `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV"` — `$ZAI_API_KEY_SECRET` is set from `${{ secrets.ZAI_API_KEY }}` in the step's `env:` block and written to GITHUB_ENV without newline-stripping.

2. `echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV"` — same issue with `$ANTHROPIC_API_KEY_SECRET` from `${{ secrets.ANTHROPIC_API_KEY }}`.

3. `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"` — `$MODEL_INPUT` is sourced from `${{ inputs.model }}` (a workflow_dispatch input, attacker-controllable). A newline in this value injects arbitrary environment variables into subsequent steps.

Fix: sanitize each value before writing, e.g.:
```bash
safe=$(printf '%s' "$MODEL_INPUT" | tr -d '\n\r')
echo "ANTHROPIC_MODEL=$safe" >> "$GITHUB_ENV"
```

Locations:

- `.github/workflows/outrider.yml:51`
- `.github/workflows/outrider.yml:53`
- `.github/workflows/outrider.yml:56`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings across action.yml, outrider.yml, and outrider-daily.yml:

1. unpinned-uses: Pinned actions/setup-python@v5 (SHA: a26af69be951a213d495a4c3e4e4022e16d87065), actions/setup-node@v4 (SHA: 49933ea5288caeca8642d1e84afbd3f7d6820020), and actions/checkout@v4 (SHA: 34e114876b0b11c390a56381ad16ebd13914f8d5) in all three files.

2. script-injection: Moved all ${{ }} expressions from run: shell strings into env: blocks. In action.yml: github.action_path moved to ACTION_PATH env var in both affected steps. In outrider.yml: inputs.provider moved to PROVIDER_INPUT, secrets.REMYX_API_KEY moved to REMYX_API_KEY_SECRET, and github.repository moved to GITHUB_REPOSITORY_VALUE.

3. github-env-injection: Added printf '%s' "$VAR" | tr -d '\n\r' sanitization before all three GITHUB_ENV writes in the 'Configure provider auth' step (ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, and MODEL_INPUT).

