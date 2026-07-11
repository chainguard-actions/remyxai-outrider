<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.7.15** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses mutable tag refs instead of pinned SHA digests: 'actions/setup-python@v5' and 'actions/setup-node@v4'. These can be silently updated to malicious versions.

Locations:

- `action.yml:337`
- `action.yml:342`

### unpinned-uses (severity: high)

.github/workflows/outrider.yml uses a mutable tag ref instead of a pinned SHA digest: 'actions/checkout@v4'. This can be silently updated to a malicious version.

Locations:

- `.github/workflows/outrider.yml:41`

### script-injection (severity: high)

Sub-rule (a): ${{ github.action_path }} is interpolated directly inside run: shell commands in action.yml. Any ${{ ... }} expression inside a run: block is a script-injection finding. Offending lines: 'install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph' and 'python ${{ github.action_path }}/src/run.py'.

Locations:

- `action.yml:356`
- `action.yml:430`

### script-injection (severity: high)

Sub-rule (a): ${{ inputs.provider }} is interpolated directly inside a run: shell command in .github/workflows/outrider.yml. An attacker-controlled workflow_dispatch input is injected into the shell command string before the shell parses it. Offending line: 'if [ "${{ inputs.provider }}" = "zai" ]; then'.

Locations:

- `.github/workflows/outrider.yml:53`

### script-injection (severity: high)

Sub-rule (a): ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} are interpolated directly inside a run: shell command in .github/workflows/outrider.yml. Offending line: 'token="$(curl -sf -X POST -H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}" ... -d "{\"repo\": \"${{ github.repository }}\"}" ...'.

Locations:

- `.github/workflows/outrider.yml:60`

### github-env-injection (severity: high)

In .github/workflows/outrider.yml, the 'Configure provider auth' step writes $MODEL_INPUT (sourced from ${{ inputs.model }}, a workflow_dispatch input) to $GITHUB_ENV without sanitization (no 'printf | tr -d newlines' step). An attacker can inject newlines into the model input to poison GITHUB_ENV with arbitrary key=value pairs. Offending line: 'echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"'.

Locations:

- `.github/workflows/outrider.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 6 findings: (1) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065 in action.yml. (2) Pinned actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020 in action.yml. (3) Pinned actions/checkout@v4 to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 in outrider.yml. (4) Fixed script injection: moved ${{ github.action_path }} into env: blocks (ACTION_PATH) in both the gh-graph install step and the recommend step in action.yml. (5) Fixed script injection: moved ${{ inputs.provider }} into env: block (PROVIDER_INPUT) in outrider.yml Configure provider auth step. (6) Fixed script injection: moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} into env: block in outrider.yml Mint Remyx bot token step. (7) Fixed github-env-injection: sanitized MODEL_INPUT with 'printf | tr -d newlines' before writing ANTHROPIC_MODEL to GITHUB_ENV.

