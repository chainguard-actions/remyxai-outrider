<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.42

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.42** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the upstream tag is moved or the repository is compromised.

In action.yml:
- `uses: actions/setup-python@v5` (tag, not SHA)
- `uses: actions/setup-node@v4` (tag, not SHA)

In .github/workflows/outrider-daily.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

In .github/workflows/outrider.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

All should be pinned to full 40-character hex commit SHAs, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:390`
- `action.yml:395`
- `.github/workflows/outrider-daily.yml:37`
- `.github/workflows/outrider.yml:55`

### script-injection (severity: high)

Multiple `run:` blocks interpolate GitHub Actions expressions (`${{ ... }}`) directly into shell command strings (sub-rule a), allowing an attacker-controlled value to be parsed as shell syntax before quoting can protect it.

**outrider.yml — "Configure provider auth" step (~line 68):**
```
if [ "${{ inputs.provider }}" = "zai" ]; then
```
`inputs.provider` is a `workflow_dispatch` input; a malicious value could break out of the string comparison and inject shell commands. Fix: move to an `env:` var and reference `"$INPUT_PROVIDER"` in the shell.

**outrider-weekly-refine.yml — "Pick candidate" step (~lines 57-59):**
```
LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"
OVERRIDE="${{ inputs.pick-override || '' }}"
OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"
```
All three `workflow_dispatch` inputs are interpolated directly into the shell script. Fix: move them to the step's `env:` block and reference via `$LOOKBACK_DAYS`, `$OVERRIDE`, `$OVERRIDE_ARXIV`.

**action.yml — "Install gh-graph selection-pass tool on PATH" step:**
```
install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph
```
`github.action_path` is a GitHub context value interpolated directly into the shell command. Fix: use the `$GITHUB_ACTION_PATH` environment variable instead.

**action.yml — "Recommend + implement + open PR" step:**
```
python ${{ github.action_path }}/src/run.py
```
Same issue — `github.action_path` interpolated directly into the shell command. Fix: use `$GITHUB_ACTION_PATH`.

Locations:

- `.github/workflows/outrider.yml:68`
- `.github/workflows/outrider-weekly-refine.yml:57`
- `action.yml:448`
- `action.yml:545`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from workflow-controlled (untrusted) sources to `$GITHUB_ENV` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A newline embedded in any of these values can inject arbitrary environment variables into subsequent steps.

**action.yml — "Configure backend from provider input" step:**
The step writes inherited composite-action env vars (set by the calling workflow) directly to `$GITHUB_ENV`:
```bash
echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"       # ZAI_API_KEY from caller
echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"  # MOONSHOT_API_KEY from caller
echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"            # inputs.model from caller
```
All three variables are caller-supplied and must be sanitized before writing to `$GITHUB_ENV`.

**outrider.yml — "Configure provider auth" step:**
The step writes env vars sourced from `${{ secrets.* }}` and `${{ inputs.model }}` to `$GITHUB_ENV` without sanitization:
```bash
echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV"
echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV"
echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"
```
While secrets are typically safe, `$MODEL_INPUT` comes from `${{ inputs.model }}` (a `workflow_dispatch` input) and is unsanitized. All writes should use the `printf '%s' "$VAR" | tr -d '\n\r'` pattern before echoing to `$GITHUB_ENV`.

Locations:

- `action.yml:487`
- `action.yml:494`
- `action.yml:502`
- `.github/workflows/outrider.yml:70`
- `.github/workflows/outrider.yml:72`
- `.github/workflows/outrider.yml:75`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings across action.yml and the three workflow files:

1. **unpinned-uses**: Pinned actions/setup-python@v5→SHA a26af69, actions/setup-node@v4→SHA 49933ea, and actions/checkout@v4→SHA 11d5960 (in both outrider-daily.yml and outrider.yml).

2. **script-injection**: (a) outrider.yml 'Configure provider auth': moved `${{ inputs.provider }}` from shell if-condition into env block as PROVIDER_INPUT; (b) outrider-weekly-refine.yml 'Pick candidate': moved all three `${{ inputs.* }}` interpolations into step env block; (c) action.yml both steps: replaced `${{ github.action_path }}` with the built-in `$GITHUB_ACTION_PATH` env var.

3. **github-env-injection** + **static-unsanitized-env-write**: Applied `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before all writes to $GITHUB_ENV in both action.yml 'Configure backend from provider input' (ZAI_API_KEY, MOONSHOT_API_KEY, INPUT_MODEL) and outrider.yml 'Configure provider auth' (ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, MODEL_INPUT).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

1. outrider.yml (script-injection): Moved `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` from the `run:` shell string of the 'Mint Remyx bot token' step into the step's `env:` block as `REMYX_API_KEY_SECRET` and `GITHUB_REPO`. The shell script now references these as plain environment variables, eliminating the script-injection risk. 2. outrider-weekly-refine.yml (github-env-injection): Added newline sanitization before writing `$BRANCH` and `$ARXIV` to `$GITHUB_OUTPUT` in the 'Pick candidate from past week's drafter output' step. Both values are now passed through `printf '%s' ... | tr -d '\n\r'` before being written, preventing injection of additional key=value pairs via embedded newlines.

