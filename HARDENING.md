<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.44

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.44** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

In action.yml:
- `uses: actions/setup-python@v5` (tag, not SHA)
- `uses: actions/setup-node@v4` (tag, not SHA)

In .github/workflows/outrider-daily.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

In .github/workflows/outrider.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

Locations:

- `action.yml:476`
- `action.yml:480`
- `.github/workflows/outrider-daily.yml:37`
- `.github/workflows/outrider.yml:55`

### script-injection (severity: high)

Multiple run: blocks interpolate ${{ ... }} expressions directly into shell command strings (rule a), allowing an attacker-controlled value to be interpreted as shell code before the shell ever sees it.

action.yml — Install gh-graph step:
  install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph

action.yml — Recommend + implement + open PR step:
  python ${{ github.action_path }}/src/run.py

outrider.yml — Configure provider auth step:
  if [ "${{ inputs.provider }}" = "zai" ]; then
inputs.provider is a workflow_dispatch input interpolated directly into the shell string. Although it has a type: choice constraint in the UI, the raw API bypasses that constraint.

outrider-weekly-refine.yml — Pick candidate step:
  LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"
  OVERRIDE="${{ inputs.pick-override || '' }}"
  OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"
All three workflow_dispatch inputs are interpolated directly into shell variable assignments without routing through an env: block first.

Locations:

- `action.yml:492`
- `action.yml:566`
- `.github/workflows/outrider.yml:67`
- `.github/workflows/outrider-weekly-refine.yml:52`
- `.github/workflows/outrider-weekly-refine.yml:53`
- `.github/workflows/outrider-weekly-refine.yml:54`

### github-env-injection (severity: high)

Several run: blocks write inherited process environment variables (set by the calling workflow or via env: blocks from inputs/secrets) to $GITHUB_ENV without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). A newline embedded in any of these values would allow injection of arbitrary environment variable definitions into subsequent steps.

action.yml — Configure backend from provider input step:
- echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV" — $ZAI_API_KEY is an inherited env var from the calling workflow, written unsanitized.
- echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV" — $MOONSHOT_API_KEY is an inherited env var, written unsanitized.
- echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV" — $INPUT_MODEL is set from ${{ inputs.model }} in the env: block, then written to GITHUB_ENV unsanitized.

outrider.yml — Configure provider auth step:
- echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV" — $ZAI_API_KEY_SECRET is set from ${{ secrets.ZAI_API_KEY }} in the env: block, written unsanitized.
- echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV" — $ANTHROPIC_API_KEY_SECRET is set from ${{ secrets.ANTHROPIC_API_KEY }}, written unsanitized.
- echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV" — $MODEL_INPUT is set from ${{ inputs.model }}, written unsanitized.

Locations:

- `action.yml:530`
- `action.yml:536`
- `action.yml:558`
- `.github/workflows/outrider.yml:68`
- `.github/workflows/outrider.yml:70`
- `.github/workflows/outrider.yml:73`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings:

1. **unpinned-uses**: Pinned all four action references to full SHAs:
   - action.yml: setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5
   - action.yml: setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4
   - outrider-daily.yml: checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4
   - outrider.yml: checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4

2. **script-injection**: Moved all ${{ }} expressions from run: shell strings into env: blocks:
   - action.yml Install gh-graph: github.action_path → env: ACTION_PATH
   - action.yml Recommend step: github.action_path → env: ACTION_PATH
   - outrider.yml Configure provider auth: inputs.provider → env: PROVIDER_INPUT
   - outrider-weekly-refine.yml Pick candidate: inputs.lookback-days/pick-override/pick-override-arxiv → env: block

3. **github-env-injection** + **static-unsanitized-env-write**: Replaced all bare `echo "VAR=$VALUE" >> $GITHUB_ENV` with `printf 'VAR=%s\n' "$(printf '%s' "$VALUE" | tr -d '\n\r')" >> "$GITHUB_ENV"` for ZAI_API_KEY, MOONSHOT_API_KEY, INPUT_MODEL in action.yml and ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, MODEL_INPUT in outrider.yml.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

1. outrider.yml 'Mint Remyx bot token' step: moved `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` out of the `run:` shell string into an `env:` block (as REMYX_API_KEY_SECRET and REPO), then referenced them as plain shell variables in the curl command. 2. outrider-weekly-refine.yml 'Pick candidate' step: added sanitization via `printf '%s' "$VAR" | tr -d '\n\r'` for both BRANCH and ARXIV before writing them to $GITHUB_OUTPUT, preventing newline injection from the user-controlled `pick-override` and `pick-override-arxiv` workflow_dispatch inputs.

