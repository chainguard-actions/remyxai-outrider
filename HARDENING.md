<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.48

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.48** was hardened automatically. 11 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses mutable version tags instead of pinned SHA digests: `actions/setup-python@v5` and `actions/setup-node@v4`. These can be silently updated to malicious versions.

Locations:

- `action.yml:236`
- `action.yml:241`

### unpinned-uses (severity: high)

.github/workflows/outrider-daily.yml uses `actions/checkout@v4` — a mutable version tag instead of a pinned 40-character SHA digest.

Locations:

- `.github/workflows/outrider-daily.yml:37`

### unpinned-uses (severity: high)

.github/workflows/outrider.yml uses `actions/checkout@v4` — a mutable version tag instead of a pinned 40-character SHA digest.

Locations:

- `.github/workflows/outrider.yml:65`

### script-injection (severity: high)

Rule (a): `${{ github.action_path }}` is directly interpolated inside the `run:` shell command string in the 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. Any `${{ ... }}` expression directly inside a run: script is a script-injection finding.

Locations:

- `action.yml:265`

### script-injection (severity: high)

Rule (a): `${{ github.action_path }}` is directly interpolated inside the `run:` shell command string in the 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py`. Any `${{ ... }}` expression directly inside a run: script is a script-injection finding.

Locations:

- `action.yml:399`

### script-injection (severity: high)

Rule (a): Attacker-controlled `inputs.*` values are directly interpolated inside the `run:` shell command string in the 'Pick candidate from past week's drafter output' step: `LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"`, `OVERRIDE="${{ inputs.pick-override || '' }}"`, `OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"`. A malicious `workflow_dispatch` input can inject arbitrary shell commands.

Locations:

- `.github/workflows/outrider-weekly-refine.yml:58`
- `.github/workflows/outrider-weekly-refine.yml:59`
- `.github/workflows/outrider-weekly-refine.yml:60`

### script-injection (severity: high)

Rule (a): `${{ inputs.provider }}` is directly interpolated inside the `run:` shell command string in the 'Configure provider auth' step: `if [ "${{ inputs.provider }}" = "zai" ]; then`. A malicious `workflow_dispatch` input can inject arbitrary shell commands.

Locations:

- `.github/workflows/outrider.yml:78`

### script-injection (severity: high)

Rule (a): `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` are directly interpolated inside the `run:` shell command string in the 'Mint Remyx bot token' step: `token="$(curl -sf -X POST -H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}" -d "{\"repo\": \"${{ github.repository }}\"}" ...)"`. Any `${{ ... }}` expression directly inside a run: script is a script-injection finding.

Locations:

- `.github/workflows/outrider.yml:90`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes `$INPUT_MODEL` (sourced from `inputs.model` via the env block) to `$GITHUB_ENV` without sanitization: `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"`. An attacker-controlled model name containing newlines could inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:393`

### github-env-injection (severity: high)

The 'Configure provider auth' step writes `$MODEL_INPUT` (sourced from `inputs.model` via the env block) to `$GITHUB_ENV` without sanitization: `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"`. An attacker-controlled model name containing newlines could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/outrider.yml:84`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all 11 findings across 4 files:

1. action.yml - Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 and actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
2. action.yml - Fixed script-injection in 'Install gh-graph' step: moved ${{ github.action_path }} to env block as ACTION_PATH
3. action.yml - Fixed script-injection in 'Recommend + implement + open PR' step: moved ${{ github.action_path }} to env block as ACTION_PATH
4. action.yml - Fixed github-env-injection: sanitized INPUT_MODEL with printf/tr before writing ANTHROPIC_MODEL to $GITHUB_ENV
5. outrider-daily.yml - Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
6. outrider.yml - Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
7. outrider.yml - Fixed script-injection in 'Configure provider auth' step: moved ${{ inputs.provider }} to env block as PROVIDER_INPUT
8. outrider.yml - Fixed github-env-injection: sanitized MODEL_INPUT with printf/tr before writing ANTHROPIC_MODEL to $GITHUB_ENV
9. outrider.yml - Fixed script-injection in 'Mint Remyx bot token' step: moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} to env block as REMYX_API_KEY_SECRET and GITHUB_REPOSITORY_NAME
10. outrider-weekly-refine.yml - Fixed script-injection in 'Pick candidate' step: moved ${{ inputs.lookback-days }}, ${{ inputs.pick-override }}, and ${{ inputs.pick-override-arxiv }} to env block as INPUT_LOOKBACK_DAYS, INPUT_OVERRIDE, INPUT_OVERRIDE_ARXIV

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection vulnerabilities in the 'Configure backend from provider input' step of action.yml:
1. Line ~507 (zai case): Added `safe_zai_key=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r')` before writing to $GITHUB_ENV, and used `$safe_zai_key` in the echo.
2. Line ~517 (moonshot case): Added `safe_moonshot_key=$(printf '%s' "$MOONSHOT_API_KEY" | tr -d '\n\r')` before writing to $GITHUB_ENV, and used `$safe_moonshot_key` in the echo.
The $INPUT_MODEL write was already correctly sanitized and was not modified.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/outrider-weekly-refine.yml at the 'Pick candidate from past week's drafter output' step. The $BRANCH variable (attacker-controllable via inputs.pick-override) and $ARXIV variable (attacker-controllable via inputs.pick-override-arxiv) were being written directly to $GITHUB_OUTPUT without sanitization. Fixed by introducing safe_branch and safe_arxiv variables that strip newlines/carriage returns using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

