<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.7.13** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses actions/setup-python@v5 and actions/setup-node@v4 — both are mutable tag references, not pinned to a full 40-character commit SHA. A compromised or updated tag could silently introduce malicious code.

Locations:

- `action.yml:271`
- `action.yml:276`

### unpinned-uses (severity: high)

.github/workflows/outrider.yml uses actions/checkout@v4 — a mutable tag reference, not pinned to a full 40-character commit SHA. A compromised or updated tag could silently introduce malicious code.

Locations:

- `.github/workflows/outrider.yml:38`

### script-injection (severity: high)

Sub-rule (a): ${{ github.action_path }} is interpolated directly inside two run: shell command strings in action.yml. Any ${{ ... }} expression directly inside a run: block is a script-injection risk because YAML template substitution occurs before the shell ever sees the value. Offending lines: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` and `python ${{ github.action_path }}/src/run.py`.

Locations:

- `action.yml:296`
- `action.yml:393`

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ ... }} expressions are interpolated directly inside run: shell command strings in .github/workflows/outrider.yml. (1) `if [ "${{ inputs.provider }}" = "zai" ]` — attacker-controlled workflow_dispatch input injected into shell. (2) `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` interpolated directly in the curl command string in the Mint Remyx bot token step.

Locations:

- `.github/workflows/outrider.yml:51`
- `.github/workflows/outrider.yml:63`

### github-env-injection (severity: high)

In .github/workflows/outrider.yml, the Configure provider auth step writes $MODEL_INPUT (sourced from ${{ inputs.model }}, a workflow_dispatch input) to $GITHUB_ENV without sanitization: `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"`. An attacker could inject newlines into the model input to set arbitrary environment variables for subsequent steps.

Locations:

- `.github/workflows/outrider.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 5 findings across action.yml and .github/workflows/outrider.yml:

1. **unpinned-uses (action.yml)**: Pinned `actions/setup-python@v5` → `@a26af69be951a213d495a4c3e4e4022e16d87065 # v5` and `actions/setup-node@v4` → `@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`.

2. **unpinned-uses (outrider.yml)**: Pinned `actions/checkout@v4` → `@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4`.

3. **script-injection (action.yml)**: Moved both `${{ github.action_path }}` expressions out of `run:` shell strings into `env:` blocks as `ACTION_PATH`, then referenced `$ACTION_PATH` in the shell commands for the gh-graph install step and the main recommend step.

4. **script-injection (outrider.yml)**: (a) Moved `${{ inputs.provider }}` into the step's `env:` block as `PROVIDER_INPUT` and replaced the direct interpolation in the `if` condition with `$PROVIDER_INPUT`. (b) Moved `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` into the Mint Remyx bot token step's `env:` block as `REMYX_API_KEY_SECRET` and `GITHUB_REPOSITORY_VALUE`, replacing direct interpolation in the curl command.

5. **github-env-injection (outrider.yml)**: Added newline sanitization for `MODEL_INPUT` before writing to `$GITHUB_ENV` using `safe_model=$(printf '%s' "$MODEL_INPUT" | tr -d '\n\r')` and writing `$safe_model` instead.

