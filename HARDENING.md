<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.16

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.7.16** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple action references use mutable version tags instead of pinned SHA digests, making the action vulnerable to supply-chain attacks. In action.yml: 'actions/setup-python@v5' and 'actions/setup-node@v4'. In .github/workflows/outrider.yml and outrider-daily.yml: 'actions/checkout@v4'.

Locations:

- `action.yml:338`
- `action.yml:342`
- `.github/workflows/outrider.yml:41`
- `.github/workflows/outrider-daily.yml:29`

### script-injection (severity: high)

Sub-rule (a): ${{ ... }} expressions are interpolated directly inside run: shell command strings. In action.yml: '${{ github.action_path }}' is used directly in two run: blocks — 'install -m 0755 "${{ github.action_path }}/src/gh_graph.py"' and 'python ${{ github.action_path }}/src/run.py'. In outrider.yml: '${{ inputs.provider }}' is interpolated directly in a shell conditional ('if [ "${{ inputs.provider }}" = "zai" ]'), and '${{ github.repository }}' is interpolated directly in a curl -d argument ('-d "{\"repo\": \"${{ github.repository }}\"}"').

Locations:

- `action.yml:390`
- `action.yml:490`
- `.github/workflows/outrider.yml:53`
- `.github/workflows/outrider.yml:65`

### github-env-injection (severity: high)

In outrider.yml, the 'Configure provider auth' step sets MODEL_INPUT from '${{ inputs.model }}' in the env: block, then writes it to $GITHUB_ENV without sanitization: 'echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"'. An attacker-controlled inputs.model value containing newlines could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/outrider.yml:59`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings: (1) unpinned-uses: pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065, actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, and actions/checkout@v4 to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 in action.yml, outrider.yml, and outrider-daily.yml. (2) script-injection: moved ${{ github.action_path }} into env: blocks (ACTION_PATH) in both affected run: steps in action.yml; moved ${{ inputs.provider }} into PROVIDER_INPUT env var and ${{ github.repository }} into GITHUB_REPOSITORY_VALUE env var in outrider.yml. (3) github-env-injection: sanitized MODEL_INPUT before writing to $GITHUB_ENV using 'safe_model=$(printf '%s' "$MODEL_INPUT" | tr -d '\n\r')' to prevent newline injection attacks.

