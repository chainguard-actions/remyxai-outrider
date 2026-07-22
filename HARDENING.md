<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.37

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.37** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the upstream tag is moved.

Failing references:
- action.yml: `uses: actions/setup-python@v5`
- action.yml: `uses: actions/setup-node@v4`
- .github/workflows/outrider-daily.yml: `uses: actions/checkout@v4`
- .github/workflows/outrider.yml: `uses: actions/checkout@v4`

Locations:

- `action.yml:474`
- `action.yml:479`
- `.github/workflows/outrider-daily.yml:36`
- `.github/workflows/outrider.yml:57`

### script-injection (severity: high)

Multiple `run:` blocks interpolate `${{ ... }}` expressions directly into shell command strings (sub-rule a). GitHub Actions substitutes these expressions as raw text before the shell executes the script, allowing attacker-controlled values to inject arbitrary shell commands.

outrider.yml 'Configure provider auth' step: `if [ "${{ inputs.provider }}" = "zai" ]; then` — inputs.provider is a workflow_dispatch input interpolated directly into the shell if-condition.

outrider.yml 'Mint Remyx bot token' step: `curl ... -H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}" -d "{\"repo\": \"${{ github.repository }}\"}"` — both expressions are interpolated directly into the shell command.

outrider-weekly-refine.yml 'Pick candidate' step: `LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"`, `OVERRIDE="${{ inputs.pick-override || '' }}"`, `OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"` — all three workflow_dispatch inputs are interpolated directly into shell variable assignments.

action.yml 'Install gh-graph' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` — github.action_path flows through YAML template substitution before the shell sees it.

action.yml 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py` — same issue.

Locations:

- `.github/workflows/outrider.yml:66`
- `.github/workflows/outrider.yml:73`
- `.github/workflows/outrider-weekly-refine.yml:57`
- `.github/workflows/outrider-weekly-refine.yml:58`
- `.github/workflows/outrider-weekly-refine.yml:59`
- `action.yml:490`
- `action.yml:560`

### github-env-injection (severity: high)

Several `run:` blocks write values derived from workflow-controlled sources to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines into these values to add arbitrary environment variables visible to all subsequent steps.

action.yml 'Configure backend from provider input' step writes inherited composite-action env vars to GITHUB_ENV without sanitization:
  echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"  (ZAI_API_KEY is caller-supplied)
  echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"  (MOONSHOT_API_KEY is caller-supplied)
  echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"  (INPUT_MODEL is set from inputs.model)

outrider.yml 'Configure provider auth' step writes env vars sourced from secrets/inputs to GITHUB_ENV without sanitization:
  echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV"
  echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV"
  echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"  (MODEL_INPUT set from inputs.model)

Routing through env: does not sanitize values; printf '%s' "$VAR" | tr -d '\n\r' is required before each write.

Locations:

- `action.yml:516`
- `action.yml:522`
- `action.yml:527`
- `action.yml:533`
- `action.yml:538`
- `action.yml:543`
- `.github/workflows/outrider.yml:67`
- `.github/workflows/outrider.yml:69`
- `.github/workflows/outrider.yml:72`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:636`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings across action.yml and .github/workflows/:

1. unpinned-uses: Pinned actions/setup-python@v5 → SHA a26af69..., actions/setup-node@v4 → SHA 49933ea..., and actions/checkout@v4 → SHA 11d5960... in all three workflow files.

2. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks. In outrider.yml, moved inputs.provider to PROVIDER_INPUT and secrets/github.repository to env vars in the Mint step. In outrider-weekly-refine.yml, moved all three inputs to LOOKBACK_DAYS_INPUT/OVERRIDE_INPUT/OVERRIDE_ARXIV_INPUT. In action.yml, moved github.action_path to ACTION_PATH env var in both the gh-graph install step and the Recommend step.

3. github-env-injection + static-unsanitized-env-write: Added printf '%s' "$VAR" | tr -d '\n\r' sanitization before every write to $GITHUB_ENV for caller-supplied values. In action.yml's Configure backend step: sanitized ZAI_API_KEY, MOONSHOT_API_KEY, and INPUT_MODEL. In outrider.yml's Configure provider auth step: sanitized ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, and MODEL_INPUT.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection vulnerabilities in .github/workflows/outrider-weekly-refine.yml:

1. Added `safe_branch=$(printf '%s' "$BRANCH" | tr -d '\n\r')` before writing BRANCH to $GITHUB_OUTPUT, and changed the write to use `safe_branch` instead of `$BRANCH`.

2. Added `safe_arxiv=$(printf '%s' "$ARXIV" | tr -d '\n\r')` before writing ARXIV to $GITHUB_OUTPUT, and changed the write to use `safe_arxiv` instead of `$ARXIV`.

3. Also changed the arxiv format validation grep from `-E` (ERE, where `^`/`$` match line boundaries) to `-P` (PCRE, where `^`/`$` match string boundaries by default), preventing a newline-containing value like `2301.00001\nmalicious=injected` from passing the format check on its first line while still injecting content.

