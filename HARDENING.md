<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.7.21** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks. In action.yml: `actions/setup-python@v5` and `actions/setup-node@v4`. In .github/workflows/outrider.yml and outrider-daily.yml: `actions/checkout@v4`.

Locations:

- `action.yml:306`
- `action.yml:311`
- `.github/workflows/outrider.yml:41`
- `.github/workflows/outrider-daily.yml:29`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings. In .github/workflows/outrider.yml, the 'Configure provider auth' step uses `${{ inputs.provider }}` directly in a shell conditional: `if [ "${{ inputs.provider }}" = "zai" ]`. The 'Mint Remyx bot token' step uses `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` directly in a curl command. In action.yml, the 'Install gh-graph' step uses `${{ github.action_path }}` directly in a shell command, and the final 'Recommend + implement + open PR' step uses `${{ github.action_path }}` in `python ${{ github.action_path }}/src/run.py`.

Locations:

- `.github/workflows/outrider.yml:54`
- `.github/workflows/outrider.yml:66`
- `action.yml:493`
- `action.yml:651`

### github-env-injection (severity: high)

In .github/workflows/outrider.yml, the 'Configure provider auth' step writes `$MODEL_INPUT` (sourced from `inputs.model` via `MODEL_INPUT: ${{ inputs.model }}` in the env block) to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`): `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"`. An attacker controlling the `model` workflow_dispatch input could inject newlines to set arbitrary environment variables.

Locations:

- `.github/workflows/outrider.yml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

1. unpinned-uses: Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 and actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 in action.yml; pinned actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5 in both .github/workflows/outrider.yml and outrider-daily.yml. 2. script-injection: In action.yml, moved github.action_path into env vars (ACTION_PATH) in both the 'Install gh-graph' step and the 'Recommend + implement + open PR' step. In outrider.yml, moved inputs.provider into PROVIDER_INPUT env var (fixing the shell conditional), and moved secrets.REMYX_API_KEY and github.repository into env vars (REMYX_API_KEY_SECRET, GITHUB_REPOSITORY_VALUE) in the 'Mint Remyx bot token' step. 3. github-env-injection: In outrider.yml 'Configure provider auth' step, sanitized MODEL_INPUT before writing to GITHUB_ENV using printf '%s' ... | tr -d '\n\r' to strip newlines.

