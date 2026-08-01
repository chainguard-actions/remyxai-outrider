<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.45

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.45** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

In action.yml:
- `uses: actions/setup-python@v5` (tag ref)
- `uses: actions/setup-node@v4` (tag ref)

In .github/workflows/outrider-daily.yml:
- `uses: actions/checkout@v4` (tag ref)

In .github/workflows/outrider.yml:
- `uses: actions/checkout@v4` (tag ref)

All should be pinned to their full SHA digest, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:336`
- `action.yml:341`
- `.github/workflows/outrider-daily.yml:38`
- `.github/workflows/outrider.yml:55`

### script-injection (severity: high)

Multiple `run:` blocks interpolate GitHub Actions expressions (`${{ ... }}`) directly into shell command strings, enabling script injection.

**action.yml — "Install gh-graph selection-pass tool on PATH" step (sub-rule a):**
```
run: |
  install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph
```
`${{ github.action_path }}` is a `github.*` context value injected directly into the shell command.

**action.yml — "Recommend + implement + open PR" step (sub-rule a):**
```
run: |
  python ${{ github.action_path }}/src/run.py
```
Same issue — `${{ github.action_path }}` directly in the shell command, unquoted.

**outrider.yml — "Configure provider auth" step (sub-rule a):**
```
run: |
  if [ "${{ inputs.provider }}" = "zai" ]; then
```
`${{ inputs.provider }}` is a workflow_dispatch input interpolated directly into the shell `if` condition, allowing an attacker to inject shell metacharacters.

**outrider.yml — "Mint Remyx bot token" step (sub-rule a):**
```
run: |
  token="$(curl -sf -X POST \
    -H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}" \
    -d "{\"repo\": \"${{ github.repository }}\"}" ...)"
```
`${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` are interpolated directly into the shell command string.

**outrider-weekly-refine.yml — "Pick candidate" step (sub-rule a):**
```
run: |
  LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"
  OVERRIDE="${{ inputs.pick-override || '' }}"
  OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"
```
Three `workflow_dispatch` inputs are interpolated directly into shell variable assignments, allowing injection of shell metacharacters via the input values.

Locations:

- `action.yml:360`
- `action.yml:497`
- `.github/workflows/outrider.yml:64`
- `.github/workflows/outrider.yml:74`
- `.github/workflows/outrider-weekly-refine.yml:57`

### github-env-injection (severity: high)

Multiple `run:` blocks write inherited process environment variables (set by the calling workflow) to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`), enabling newline injection that can poison subsequent steps' environment.

**action.yml — "Configure backend from provider input" step:**
The variables `$ZAI_API_KEY`, `$MOONSHOT_API_KEY`, and `$INPUT_MODEL` are inherited from the calling workflow's `env:` block and written directly to `$GITHUB_ENV`:
```bash
echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"      # FAIL: no tr -d newlines
echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV" # FAIL: no tr -d newlines
echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"           # FAIL: no tr -d newlines
```
A calling workflow can set `ZAI_API_KEY`, `MOONSHOT_API_KEY`, or `model` input to a value containing newlines, injecting arbitrary key=value pairs into `$GITHUB_ENV`.

**outrider.yml — "Configure provider auth" step:**
The variables `$ZAI_API_KEY_SECRET`, `$ANTHROPIC_API_KEY_SECRET`, and `$MODEL_INPUT` (sourced from `inputs.model` via env) are written to `$GITHUB_ENV` without sanitization:
```bash
echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV"  # FAIL: no tr -d newlines
echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV" # FAIL: no tr -d newlines
echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"               # FAIL: no tr -d newlines
```
`MODEL_INPUT` is set from `${{ inputs.model }}` (a workflow_dispatch input) and written to `$GITHUB_ENV` without sanitization.

Locations:

- `action.yml:430`
- `.github/workflows/outrider.yml:65`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all 4 findings:

1. unpinned-uses: Pinned actions/setup-python@v5→SHA, actions/setup-node@v4→SHA, and actions/checkout@v4→SHA (in both outrider.yml and outrider-daily.yml) using full 40-char commit SHAs with tag comments.

2. script-injection: Moved all ${{ }} expressions from run: blocks to env: blocks:
   - action.yml gh-graph step: ACTION_PATH env var for github.action_path
   - action.yml recommend step: ACTION_PATH env var for github.action_path
   - outrider.yml Configure provider auth: PROVIDER_INPUT env var for inputs.provider
   - outrider.yml Mint Remyx bot token: REMYX_API_KEY_SECRET and GITHUB_REPOSITORY_VALUE env vars
   - outrider-weekly-refine.yml Pick candidate: INPUT_LOOKBACK_DAYS, INPUT_OVERRIDE, INPUT_OVERRIDE_ARXIV env vars

3. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization for all GITHUB_ENV writes in action.yml (ZAI_API_KEY, MOONSHOT_API_KEY, INPUT_MODEL) and outrider.yml (ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, MODEL_INPUT).

4. static-unsanitized-env-write: The INPUT_MODEL write in action.yml's Configure backend step is now sanitized (same fix as github-env-injection above).

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in outrider-weekly-refine.yml at lines 120-121. Before writing BRANCH and ARXIV to $GITHUB_OUTPUT, both values are now sanitized using `printf '%s' "$VAR" | tr -d '\n\r'` to strip newline characters. This prevents an attacker-controlled workflow_dispatch input (pick-override or pick-override-arxiv) containing a newline from injecting extra key-value pairs into the step output file.

