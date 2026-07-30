<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.41

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.41** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ }}` expressions are interpolated directly inside `run:` shell command strings in action.yml. (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` — `github.action_path` flows through YAML template substitution before the shell sees it. (2) `python ${{ github.action_path }}/src/run.py` — same issue. Any `${{ ... }}` directly inside a `run:` script is a script-injection finding regardless of which context it reads from.

Locations:

- `action.yml:396`
- `action.yml:530`

### script-injection (severity: high)

Sub-rule (a): `${{ }}` expressions are interpolated directly inside `run:` shell command strings in outrider.yml. (1) In the 'Configure provider auth' step: `if [ "${{ inputs.provider }}" = "zai" ]` — direct interpolation of `inputs.provider` in a shell command. (2) In the 'Mint Remyx bot token' step: `-H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}"` and `-d "{\"repo\": \"${{ github.repository }}\"}"` — `${{ secrets.* }}` and `${{ github.* }}` expressions directly in a run block.

Locations:

- `.github/workflows/outrider.yml:66`
- `.github/workflows/outrider.yml:75`

### script-injection (severity: high)

Sub-rule (a): `${{ }}` expressions are interpolated directly inside a `run:` shell command string in outrider-weekly-refine.yml. In the 'Pick candidate' step: `LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"`, `OVERRIDE="${{ inputs.pick-override || '' }}"`, and `OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"` — direct interpolation of `inputs.*` values into shell variable assignments. An attacker-controlled `workflow_dispatch` input could inject shell metacharacters.

Locations:

- `.github/workflows/outrider-weekly-refine.yml:55`
- `.github/workflows/outrider-weekly-refine.yml:56`
- `.github/workflows/outrider-weekly-refine.yml:57`

### github-env-injection (severity: high)

In action.yml's 'Configure backend from provider input' step, inherited process env vars `$ZAI_API_KEY` and `$MOONSHOT_API_KEY` (set by the calling workflow, not by this run block) are written to $GITHUB_ENV without the required sanitization (`printf '%s' ... | tr -d '\n\r'`): `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` and `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"`. Additionally, `INPUT_MODEL` (sourced from `inputs.model` via the step's `env:` block) is written unsanitized: `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"`. A newline in any of these values could inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:468`
- `action.yml:477`
- `action.yml:487`

### github-env-injection (severity: high)

In outrider.yml's 'Configure provider auth' step, `MODEL_INPUT` (sourced from `inputs.model` via the step's `env:` block) is written to $GITHUB_ENV without sanitization: `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"`. A newline embedded in the `inputs.model` workflow_dispatch value could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/outrider.yml:71`

### github-env-injection (severity: high)

In outrider-weekly-refine.yml's 'Pick candidate' step, `BRANCH` (derived from `$OVERRIDE` which is set from `${{ inputs.pick-override || '' }}` interpolated directly in the run block) and `ARXIV` (derived from `$OVERRIDE_ARXIV` which is set from `${{ inputs.pick-override-arxiv || '' }}`) are written to $GITHUB_OUTPUT without sanitization: `echo "picked=$BRANCH" >> "$GITHUB_OUTPUT"` and `echo "arxiv=$ARXIV" >> "$GITHUB_OUTPUT"`. A newline in a workflow_dispatch input could inject arbitrary output variables.

Locations:

- `.github/workflows/outrider-weekly-refine.yml:103`
- `.github/workflows/outrider-weekly-refine.yml:104`

### unpinned-uses (severity: high)

action.yml references external actions using mutable version tags instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the tag is moved: `uses: actions/setup-python@v5` and `uses: actions/setup-node@v4`.

Locations:

- `action.yml:381`
- `action.yml:386`

### unpinned-uses (severity: high)

outrider-daily.yml references an external action using a mutable version tag instead of a full 40-character commit SHA: `uses: actions/checkout@v4`.

Locations:

- `.github/workflows/outrider-daily.yml:38`

### unpinned-uses (severity: high)

outrider.yml references an external action using a mutable version tag instead of a full 40-character commit SHA: `uses: actions/checkout@v4`.

Locations:

- `.github/workflows/outrider.yml:57`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, static-unsanitized-env-write

**Notes:**

Fixed all 10 findings across 4 files:

**action.yml:**
- Pinned `actions/setup-python@v5` → SHA `a26af69be951a213d495a4c3e4e4022e16d87065`
- Pinned `actions/setup-node@v4` → SHA `49933ea5288caeca8642d1e84afbd3f7d6820020`
- Moved `${{ github.action_path }}` in gh-graph install step into `env: ACTION_PATH:` block
- Moved `${{ github.action_path }}` in run.py step into the step's `env:` block as `ACTION_PATH`
- Sanitized `$ZAI_API_KEY` and `$MOONSHOT_API_KEY` writes to `$GITHUB_ENV` with `printf '%s' ... | tr -d '\n\r'`
- Sanitized `$INPUT_MODEL` write to `$GITHUB_ENV` with `printf '%s' ... | tr -d '\n\r'`

**outrider.yml:**
- Pinned `actions/checkout@v4` → SHA `11d5960a326750d5838078e36cf38b85af677262`
- Moved `${{ inputs.provider }}` into `env: PROVIDER_INPUT:` block (script-injection fix)
- Moved `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` into `env:` block (script-injection fix)
- Sanitized `$MODEL_INPUT` write to `$GITHUB_ENV` with `printf '%s' ... | tr -d '\n\r'`

**outrider-daily.yml:**
- Pinned `actions/checkout@v4` → SHA `11d5960a326750d5838078e36cf38b85af677262`

**outrider-weekly-refine.yml:**
- Moved `${{ inputs.lookback-days || '7' }}`, `${{ inputs.pick-override || '' }}`, and `${{ inputs.pick-override-arxiv || '' }}` into `env:` block (script-injection fix)
- Sanitized `$BRANCH` and `$ARXIV` writes to `$GITHUB_OUTPUT` with `printf '%s' ... | tr -d '\n\r'`

