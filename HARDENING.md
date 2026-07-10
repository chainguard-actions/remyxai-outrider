<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.14

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.7.14** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references actions/setup-python@v5 and actions/setup-node@v4 using mutable version tags instead of pinned 40-character commit SHAs. These can be silently updated to malicious versions.

Locations:

- `action.yml:271`
- `action.yml:276`

### unpinned-uses (severity: high)

.github/workflows/outrider.yml references actions/checkout@v4 using a mutable version tag instead of a pinned 40-character commit SHA. This is vulnerable to supply-chain attacks.

Locations:

- `.github/workflows/outrider.yml:36`

### script-injection (severity: high)

Sub-rule (a): action.yml directly interpolates ${{ github.action_path }} inside run: shell commands in two steps. Any ${{ ... }} expression inside a run: block is a script-injection risk as the value is substituted before the shell parses the command. Offending lines: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` and `python ${{ github.action_path }}/src/run.py`.

Locations:

- `action.yml:299`
- `action.yml:358`

### script-injection (severity: high)

Sub-rule (a): .github/workflows/outrider.yml directly interpolates ${{ inputs.provider }} inside a run: shell command in the 'Configure provider auth' step: `if [ "${{ inputs.provider }}" = "zai" ]`. The inputs.provider value is attacker-controllable via workflow_dispatch and is substituted directly into the shell command before execution.

Locations:

- `.github/workflows/outrider.yml:50`

### github-env-injection (severity: high)

In .github/workflows/outrider.yml, the 'Configure provider auth' step writes the env var MODEL_INPUT (sourced from ${{ inputs.model }}, a workflow_dispatch input) to $GITHUB_ENV without sanitization: `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"`. An attacker could inject newlines to set arbitrary environment variables for subsequent steps.

Locations:

- `.github/workflows/outrider.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 5 findings: (1) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065 in action.yml. (2) Pinned actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020 in action.yml. (3) Pinned actions/checkout@v4 to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 in outrider.yml. (4) Fixed script injection in action.yml: moved ${{ github.action_path }} into env vars (ACTION_PATH) in both the gh-graph install step and the recommend step. (5) Fixed script injection in outrider.yml: moved ${{ inputs.provider }} into env var PROVIDER_INPUT and replaced direct interpolation in the if-condition. (6) Fixed github-env-injection: sanitized MODEL_INPUT with printf | tr -d '\n\r' before writing ANTHROPIC_MODEL to $GITHUB_ENV.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed both high-severity findings in .github/workflows/outrider.yml: (1) script-injection: moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} out of the 'Mint Remyx bot token' run: block into step-level env: variables (REMYX_API_KEY_SECRET and GITHUB_REPOSITORY_NAME), referencing them as plain shell variables in the script; (2) github-env-injection: added printf '%s' "$VAR" | tr -d '\n\r' sanitization before writing ZAI_API_KEY_SECRET and ANTHROPIC_API_KEY_SECRET to $GITHUB_ENV in the 'Configure provider auth' step.

