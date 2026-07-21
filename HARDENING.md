<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.38

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.38** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved.

Failing references:
- `action.yml`: `uses: actions/setup-python@v5` and `uses: actions/setup-node@v4`
- `.github/workflows/outrider-daily.yml`: `uses: actions/checkout@v4`
- `.github/workflows/outrider.yml`: `uses: actions/checkout@v4`

All should be pinned to full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:563`
- `action.yml:568`
- `.github/workflows/outrider-daily.yml:41`
- `.github/workflows/outrider.yml:61`

### script-injection (severity: high)

Sub-rule (a): `${{ ... }}` expressions are directly interpolated inside `run:` shell scripts, allowing YAML template substitution to inject arbitrary shell commands before the shell ever parses the string.

outrider.yml 'Configure provider auth' step: `if [ "${{ inputs.provider }}" = "zai" ]; then` — inputs.provider is a workflow_dispatch input an attacker can control.

outrider.yml 'Mint Remyx bot token' step: `-H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}"` and `-d "{\"repo\": \"${{ github.repository }}\"}"` are interpolated directly in the run block.

outrider-weekly-refine.yml 'Pick candidate' step: `LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"`, `OVERRIDE="${{ inputs.pick-override || '' }}"`, `OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"` — all three are workflow_dispatch inputs interpolated directly into the shell. An attacker-controlled pick-override value could inject shell commands.

action.yml 'Install gh-graph' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py"` — github.action_path interpolated directly in run block.

action.yml 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py` — github.action_path interpolated directly in run block.

Fix: move all values to env: vars and reference them as quoted shell variables (e.g. "$INPUT_PROVIDER").

Locations:

- `.github/workflows/outrider.yml:74`
- `.github/workflows/outrider.yml:86`
- `.github/workflows/outrider-weekly-refine.yml:56`
- `.github/workflows/outrider-weekly-refine.yml:57`
- `.github/workflows/outrider-weekly-refine.yml:58`
- `action.yml:563`
- `action.yml:700`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs or caller-supplied env vars to $GITHUB_ENV or $GITHUB_OUTPUT without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). A newline embedded in the value can inject arbitrary environment variable definitions into subsequent steps.

action.yml 'Configure backend from provider input' step:
- `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — $ZAI_API_KEY is an inherited env var set by the calling workflow (untrusted composite-action input path).
- `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — same issue with $MOONSHOT_API_KEY.
- `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — $INPUT_MODEL is sourced from inputs.model (caller-controlled).

outrider.yml 'Configure provider auth' step:
- `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"` — $MODEL_INPUT is sourced from inputs.model (workflow_dispatch input, caller-controlled), written without sanitization.

outrider-weekly-refine.yml 'Pick candidate' step:
- `echo "picked=$BRANCH" >> "$GITHUB_OUTPUT"` — $BRANCH is set from inputs.pick-override (workflow_dispatch input).
- `echo "arxiv=$ARXIV" >> "$GITHUB_OUTPUT"` — $ARXIV is set from inputs.pick-override-arxiv (workflow_dispatch input).

Fix: apply `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before every write to a special environment file when the source is caller-controlled.

Locations:

- `action.yml:614`
- `action.yml:622`
- `action.yml:628`
- `.github/workflows/outrider.yml:80`
- `.github/workflows/outrider-weekly-refine.yml:120`
- `.github/workflows/outrider-weekly-refine.yml:121`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all 4 findings across action.yml and 3 workflow files:

1. unpinned-uses: Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065, actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, and actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 (in both outrider-daily.yml and outrider.yml).

2. script-injection: Moved all ${{ }} expressions from run: blocks to env: blocks - github.action_path in two action.yml steps, inputs.provider in outrider.yml Configure step, secrets.REMYX_API_KEY and github.repository in outrider.yml Mint step, and all three workflow_dispatch inputs in outrider-weekly-refine.yml Pick candidate step.

3. github-env-injection: Added printf/tr sanitization before all writes to GITHUB_ENV/GITHUB_OUTPUT from caller-controlled values: ZAI_API_KEY, MOONSHOT_API_KEY, and INPUT_MODEL in action.yml; MODEL_INPUT in outrider.yml; BRANCH and ARXIV in outrider-weekly-refine.yml.

4. static-unsanitized-env-write: Covered by the github-env-injection fix for INPUT_MODEL in action.yml's Configure backend step.

