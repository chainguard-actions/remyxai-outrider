<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.33

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.33** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or overwritten.

In action.yml:
- `uses: actions/setup-python@v5` (tag, not SHA)
- `uses: actions/setup-node@v4` (tag, not SHA)

In .github/workflows/outrider.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

In .github/workflows/outrider-daily.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

All should be pinned to their full 40-character hex commit SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:307`
- `action.yml:311`
- `.github/workflows/outrider.yml:35`
- `.github/workflows/outrider-daily.yml:38`

### script-injection (severity: high)

GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings, allowing an attacker who controls the input values to inject arbitrary shell commands.

(a) In `.github/workflows/outrider.yml`, the `Configure provider auth` step interpolates `${{ inputs.provider }}` directly in the shell script:
```
if [ "${{ inputs.provider }}" = "zai" ]; then
```
This is a `workflow_dispatch` input that can be set by any user who can trigger the workflow. The value should be routed through an `env:` variable and then double-quoted in the shell.

(b) In `.github/workflows/outrider-weekly-refine.yml`, the `Pick candidate` step interpolates three `workflow_dispatch` inputs directly in the shell script:
```
LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"
OVERRIDE="${{ inputs.pick-override || '' }}"
OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"
```
All three should be routed through `env:` variables.

(c) In `action.yml`, the `Install gh-graph selection-pass tool on PATH` step interpolates `${{ github.action_path }}` directly in the shell script:
```
install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph
```
And the final `Recommend + implement + open PR` step interpolates `${{ github.action_path }}` in:
```
python ${{ github.action_path }}/src/run.py
```
Any `${{ ... }}` expression inside a `run:` block is a script-injection risk regardless of context.

Locations:

- `.github/workflows/outrider.yml:46`
- `.github/workflows/outrider-weekly-refine.yml:57`
- `.github/workflows/outrider-weekly-refine.yml:58`
- `.github/workflows/outrider-weekly-refine.yml:59`
- `action.yml:310`
- `action.yml:430`

### github-env-injection (severity: high)

Untrusted or workflow-controlled values are written to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`), allowing newline injection that can set arbitrary environment variables for subsequent steps.

**In `action.yml` — `Configure backend from provider input` step:**
- `ZAI_API_KEY` (an inherited process env var set by the calling workflow) is written directly to `$GITHUB_ENV`:
  ```
  echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"
  ```
- `MOONSHOT_API_KEY` (same pattern) is written directly to `$GITHUB_ENV`:
  ```
  echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"
  ```
- `INPUT_MODEL` (sourced from `${{ inputs.model }}` via env:) is written directly to `$GITHUB_ENV`:
  ```
  echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"
  ```

**In `.github/workflows/outrider.yml` — `Configure provider auth` step:**
- `ZAI_API_KEY_SECRET` (sourced from `${{ secrets.ZAI_API_KEY }}`) is written directly to `$GITHUB_ENV`:
  ```
  echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV"
  ```
- `ANTHROPIC_API_KEY_SECRET` (sourced from `${{ secrets.ANTHROPIC_API_KEY }}`) is written directly to `$GITHUB_ENV`:
  ```
  echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV"
  ```
- `MODEL_INPUT` (sourced from `${{ inputs.model }}`) is written directly to `$GITHUB_ENV`:
  ```
  echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"
  ```

All writes should be preceded by: `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` and then `echo "KEY=$safe" >> "$GITHUB_ENV"`.

Locations:

- `action.yml:393`
- `action.yml:400`
- `action.yml:407`
- `action.yml:416`
- `action.yml:425`
- `.github/workflows/outrider.yml:47`
- `.github/workflows/outrider.yml:49`
- `.github/workflows/outrider.yml:52`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:636`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings:

1. **unpinned-uses**: Pinned all mutable tag references to full 40-char SHAs:
   - action.yml: setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065, setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - outrider.yml and outrider-daily.yml: checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262

2. **script-injection**: Moved all ${{ }} expressions out of run: blocks into env: variables:
   - action.yml Install gh-graph step: github.action_path → ACTION_PATH env var
   - action.yml Recommend step: github.action_path added to existing env block as ACTION_PATH
   - outrider.yml Configure provider auth: inputs.provider → PROVIDER_INPUT env var
   - outrider-weekly-refine.yml Pick candidate: all three inputs moved to env vars (INPUT_LOOKBACK_DAYS, INPUT_OVERRIDE, INPUT_OVERRIDE_ARXIV)

3. **github-env-injection**: Added printf/tr sanitization before all GITHUB_ENV writes:
   - action.yml Configure backend step: sanitized ZAI_API_KEY, MOONSHOT_API_KEY, INPUT_MODEL
   - outrider.yml Configure provider auth step: sanitized ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, MODEL_INPUT

4. **static-unsanitized-env-write**: Covered by the github-env-injection fix for INPUT_MODEL in action.yml.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

1. outrider.yml 'Mint Remyx bot token' step: moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} out of the run: shell string into the step's env: block (as REMYX_API_KEY_SECRET and GITHUB_REPO), then referenced them as plain shell variables. 2. outrider-weekly-refine.yml 'Pick candidate' step: added sanitization of $BRANCH and $ARXIV (user-controlled inputs) using printf '%s' "$VAR" | tr -d '\n\r' before writing to $GITHUB_OUTPUT, preventing newline injection into subsequent step outputs.

