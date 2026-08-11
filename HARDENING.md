<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.50

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.50** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags rather than full 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or overwritten.

Failing references:
- action.yml: `uses: actions/setup-python@v5`
- action.yml: `uses: actions/setup-node@v4`
- .github/workflows/outrider-daily.yml: `uses: actions/checkout@v4`
- .github/workflows/outrider.yml: `uses: actions/checkout@v4`

Locations:

- `action.yml:388`
- `action.yml:393`
- `.github/workflows/outrider-daily.yml:37`
- `.github/workflows/outrider.yml:57`

### script-injection (severity: high)

Multiple `run:` blocks interpolate `${{ ... }}` expressions directly into shell command strings, violating sub-rule (a). This allows the expression value to be parsed as shell syntax before quoting can protect it.

(1) action.yml — 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` — `github.action_path` interpolated directly in run:.

(2) action.yml — 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py` — `github.action_path` interpolated directly in run:.

(3) .github/workflows/outrider.yml — 'Configure provider auth' step: `if [ "${{ inputs.provider }}" = "zai" ]` — attacker-controllable `inputs.provider` (workflow_dispatch) interpolated directly in run:.

(4) .github/workflows/outrider.yml — 'Mint Remyx bot token' step: `-H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}"` and `-d "{\"repo\": \"${{ github.repository }}\"}"` — expressions interpolated directly in run:.

(5) .github/workflows/outrider-weekly-refine.yml — 'Pick candidate from past week's drafter output' step: `LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"`, `OVERRIDE="${{ inputs.pick-override || '' }}"`, `OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"` — attacker-controllable workflow_dispatch inputs interpolated directly in run:.

Locations:

- `action.yml:410`
- `action.yml:502`
- `.github/workflows/outrider.yml:66`
- `.github/workflows/outrider.yml:80`
- `.github/workflows/outrider-weekly-refine.yml:48`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted or inherited-env sources to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`), enabling newline injection that could set arbitrary environment variables for subsequent steps.

(1) action.yml — 'Configure backend from provider input' step: `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` and `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` write inherited process env vars (set by the calling workflow) to GITHUB_ENV without sanitization. Also `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` where INPUT_MODEL is derived from `inputs.model`.

(2) .github/workflows/outrider.yml — 'Configure provider auth' step: `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV"`, `echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV"`, and `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"` (where MODEL_INPUT comes from `inputs.model`) all write to GITHUB_ENV without sanitization.

Locations:

- `action.yml:468`
- `action.yml:476`
- `action.yml:495`
- `.github/workflows/outrider.yml:67`
- `.github/workflows/outrider.yml:69`
- `.github/workflows/outrider.yml:72`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all 4 findings across action.yml and .github/workflows/ files:

1. unpinned-uses: Pinned actions/setup-python@v5, actions/setup-node@v4, and actions/checkout@v4 (×2) to full 40-char SHA digests with tag comments.

2. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks — github.action_path in two action.yml steps, inputs.provider in outrider.yml, secrets.REMYX_API_KEY + github.repository in outrider.yml Mint step, and three inputs.* in outrider-weekly-refine.yml.

3. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before all writes to $GITHUB_ENV — ZAI_API_KEY and MOONSHOT_API_KEY in action.yml Configure step, INPUT_MODEL in action.yml Configure step, and ZAI_API_KEY_SECRET/ANTHROPIC_API_KEY_SECRET/MODEL_INPUT in outrider.yml Configure step.

4. static-unsanitized-env-write: Covered by the same INPUT_MODEL sanitization fix in action.yml.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in hardened/action/.github/workflows/outrider-weekly-refine.yml. The two unsanitized writes of $BRANCH and $ARXIV to $GITHUB_OUTPUT (lines 107-108) were replaced with sanitized versions: `safe_branch=$(printf '%s' "$BRANCH" | tr -d '\n\r')` and `safe_arxiv=$(printf '%s' "$ARXIV" | tr -d '\n\r')`, followed by writing `$safe_branch` and `$safe_arxiv` to $GITHUB_OUTPUT. This prevents a workflow_dispatch caller from injecting arbitrary key=value pairs into the GITHUB_OUTPUT context via embedded newlines in the pick-override or pick-override-arxiv inputs.

