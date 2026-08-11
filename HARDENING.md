<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.51

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.51** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved.

Failing references:
- action.yml: `uses: actions/setup-python@v5`
- action.yml: `uses: actions/setup-node@v4`
- .github/workflows/outrider-daily.yml: `uses: actions/checkout@v4`
- .github/workflows/outrider.yml: `uses: actions/checkout@v4`

Locations:

- `action.yml:530`
- `action.yml:535`
- `.github/workflows/outrider-daily.yml:38`
- `.github/workflows/outrider.yml:57`

### script-injection (severity: high)

Multiple `run:` blocks interpolate `${{ ... }}` expressions directly into shell command strings (sub-rule a), bypassing shell quoting and allowing injection of shell metacharacters.

(1) action.yml — "Install gh-graph selection-pass tool on PATH" step:
  `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`
  The `github.action_path` context is interpolated directly into the shell command.

(2) action.yml — "Recommend + implement + open PR" step:
  `python ${{ github.action_path }}/src/run.py`
  Same issue — `github.action_path` interpolated directly into the shell command.

(3) .github/workflows/outrider.yml — "Configure provider auth" step:
  `if [ "${{ inputs.provider }}" = "zai" ]; then`
  The `inputs.provider` value (attacker-controllable via workflow_dispatch) is interpolated directly into the shell command string.

(4) .github/workflows/outrider-weekly-refine.yml — "Pick candidate from past week's drafter output" step:
  `LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"`
  `OVERRIDE="${{ inputs.pick-override || '' }}"`
  `OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"`
  All three workflow_dispatch inputs are interpolated directly into shell variable assignments, allowing injection of shell metacharacters.

Locations:

- `action.yml:530`
- `action.yml:760`
- `.github/workflows/outrider.yml:80`
- `.github/workflows/outrider-weekly-refine.yml:56`
- `.github/workflows/outrider-weekly-refine.yml:57`
- `.github/workflows/outrider-weekly-refine.yml:58`

### github-env-injection (severity: high)

Multiple `run:` blocks write inherited process environment variables (set by the calling workflow) to `$GITHUB_ENV` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A newline in any of these values would allow an attacker to inject arbitrary environment variable definitions into subsequent steps.

(1) action.yml — "Configure backend from provider input" step:
  - `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `ZAI_API_KEY` is an inherited env var set by the calling workflow; no sanitization applied.
  - `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — same issue with `MOONSHOT_API_KEY`.
  - `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `INPUT_MODEL` is derived from `inputs.model` (caller-controlled); no sanitization applied.

(2) .github/workflows/outrider.yml — "Configure provider auth" step:
  - `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV"` — env var from secrets, written without sanitization.
  - `echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV"` — same issue.
  - `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"` — `MODEL_INPUT` is set from `inputs.model` (caller-controlled); no sanitization applied.

Locations:

- `action.yml:665`
- `action.yml:675`
- `action.yml:700`
- `.github/workflows/outrider.yml:82`
- `.github/workflows/outrider.yml:84`
- `.github/workflows/outrider.yml:87`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all 4 findings:

1. unpinned-uses: Pinned all mutable tag references to full 40-char commit SHAs:
   - actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5 (action.yml)
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4 (action.yml)
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4 (outrider-daily.yml and outrider.yml)

2. script-injection: Moved all ${{ }} expressions from run: blocks to env: blocks:
   - action.yml 'Install gh-graph' step: github.action_path → ACTION_PATH
   - action.yml 'Recommend + implement + open PR' step: github.action_path → ACTION_PATH
   - outrider.yml 'Configure provider auth' step: inputs.provider → PROVIDER_INPUT
   - outrider-weekly-refine.yml 'Pick candidate' step: inputs.lookback-days/pick-override/pick-override-arxiv → INPUT_LOOKBACK_DAYS/INPUT_OVERRIDE/INPUT_OVERRIDE_ARXIV

3. github-env-injection + static-unsanitized-env-write: Sanitized all writes to $GITHUB_ENV using printf '%s' "$VAR" | tr -d '\n\r' pattern:
   - action.yml: ZAI_API_KEY, MOONSHOT_API_KEY, INPUT_MODEL writes
   - outrider.yml: ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, MODEL_INPUT writes

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

1. outrider.yml (Mint Remyx bot token step): Moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} from the run: shell string into the step's env: block as REMYX_API_KEY_SECRET and GITHUB_REPOSITORY_VALUE. The shell script now references these as plain environment variables, eliminating the script-injection risk from template engine substitution before shell parsing.

2. outrider-weekly-refine.yml (Pick candidate step, lines 130-131): Added newline sanitization before writing $BRANCH and $ARXIV to $GITHUB_OUTPUT. Both values are now passed through `printf '%s' "$VAR" | tr -d '\n\r'` into safe_branch and safe_arxiv variables before being echoed to $GITHUB_OUTPUT, preventing newline injection that could overwrite subsequent step outputs.

