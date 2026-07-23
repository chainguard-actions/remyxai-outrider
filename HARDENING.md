<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.21** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable version tags instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks. In action.yml: uses: actions/setup-python@v5 and uses: actions/setup-node@v4 (both tag refs). In .github/workflows/outrider.yml and outrider-daily.yml: uses: actions/checkout@v4 (tag ref).

Locations:

- `action.yml:302`
- `action.yml:307`
- `.github/workflows/outrider.yml:33`
- `.github/workflows/outrider-daily.yml:25`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions inside shell command strings (sub-rule a), causing YAML template substitution before the shell parses the command. (1) .github/workflows/outrider.yml Configure provider auth step: ${{ inputs.provider }} (workflow_dispatch input, attacker-controllable) interpolated directly in shell: if [ "${{ inputs.provider }}" = "zai" ]. (2) .github/workflows/outrider.yml Mint Remyx bot token step: ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} interpolated directly in curl command. (3) action.yml Install gh-graph step: ${{ github.action_path }} interpolated directly: install -m 0755 "${{ github.action_path }}/src/gh_graph.py". (4) action.yml Recommend + implement + open PR step: ${{ github.action_path }} interpolated directly: python ${{ github.action_path }}/src/run.py. Fix: use pre-defined environment variables such as $GITHUB_ACTION_PATH instead.

Locations:

- `.github/workflows/outrider.yml:46`
- `.github/workflows/outrider.yml:55`
- `action.yml:318`
- `action.yml:399`

### github-env-injection (severity: high)

In .github/workflows/outrider.yml, the Configure provider auth step writes $MODEL_INPUT (sourced from inputs.model, a workflow_dispatch input) to $GITHUB_ENV without sanitization. An attacker can supply a value containing newlines to inject arbitrary environment variable assignments into subsequent steps. Vulnerable code: env: MODEL_INPUT: ${{ inputs.model }} / run: echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV". Fix: safe=$(printf '%s' "$MODEL_INPUT" | tr -d '\n\r') then echo "ANTHROPIC_MODEL=$safe" >> "$GITHUB_ENV".

Locations:

- `.github/workflows/outrider.yml:51`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all findings: (1) Pinned actions/setup-python@v5, actions/setup-node@v4, and actions/checkout@v4 (×2) to full 40-char SHAs in action.yml, outrider.yml, and outrider-daily.yml. (2) Fixed four script-injection points: moved inputs.provider into PROVIDER_INPUT env var in outrider.yml; moved secrets.REMYX_API_KEY and github.repository into env vars in the Mint Remyx bot token step; replaced ${{ github.action_path }} with $GITHUB_ACTION_PATH in both action.yml run steps. (3) Fixed github-env-injection by sanitizing MODEL_INPUT with printf | tr -d '\n\r' before writing to GITHUB_ENV.

