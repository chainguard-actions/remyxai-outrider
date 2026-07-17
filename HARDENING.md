<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.14

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.14** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references actions/setup-python@v5 and actions/setup-node@v4 using mutable version tags instead of pinned 40-character commit SHAs. .github/workflows/outrider.yml references actions/checkout@v4 using a mutable version tag. Any of these could be silently replaced with a malicious version.

Locations:

- `action.yml:270`
- `action.yml:275`
- `.github/workflows/outrider.yml:43`

### script-injection (severity: high)

Sub-rule (a): ${{ ... }} expressions are interpolated directly inside run: shell command strings, allowing YAML template substitution to inject arbitrary shell code before the shell ever parses the command.

1. action.yml — 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` — ${{ github.action_path }} is interpolated directly into the shell command.

2. action.yml — 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py` — ${{ github.action_path }} is interpolated directly into the shell command.

3. .github/workflows/outrider.yml — 'Configure provider auth' step (line 56): `if [ "${{ inputs.provider }}" = "zai" ]; then` — ${{ inputs.provider }} is a workflow_dispatch input directly interpolated into the shell condition, enabling command injection by a user who can trigger the workflow.

4. .github/workflows/outrider.yml — 'Mint Remyx bot token' step (line 68): `-d "{\"repo\": \"${{ github.repository }}\"}"` — ${{ github.repository }} is interpolated directly into the curl -d argument inside the run: block.

Locations:

- `action.yml:285`
- `action.yml:380`
- `.github/workflows/outrider.yml:56`
- `.github/workflows/outrider.yml:68`

### github-env-injection (severity: high)

In .github/workflows/outrider.yml, the 'Configure provider auth' step writes the env var $MODEL_INPUT (sourced from inputs.model via `MODEL_INPUT: ${{ inputs.model }}`) to $GITHUB_ENV without sanitization:

  echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"

A workflow_dispatch caller can supply a newline-containing value for inputs.model, injecting arbitrary key=value pairs into the runner's environment for subsequent steps. The required sanitization step (`printf '%s' "$MODEL_INPUT" | tr -d '\n\r'`) is absent.

Locations:

- `.github/workflows/outrider.yml:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three finding types across action.yml and .github/workflows/outrider.yml:

1. unpinned-uses: Pinned actions/setup-python@v5 → SHA a26af69..., actions/setup-node@v4 → SHA 49933ea..., actions/checkout@v4 → SHA 34e1148... with inline # tag comments.

2. script-injection (4 locations):
   - action.yml 'Install gh-graph' step: added `ACTION_PATH: ${{ github.action_path }}` to env block, replaced `"${{ github.action_path }}/src/gh_graph.py"` with `"$ACTION_PATH/src/gh_graph.py"`.
   - action.yml 'Recommend + implement + open PR' step: added `ACTION_PATH: ${{ github.action_path }}` to existing env block, replaced `python ${{ github.action_path }}/src/run.py` with `python "$ACTION_PATH/src/run.py"`.
   - outrider.yml 'Configure provider auth' step: added `PROVIDER_INPUT: ${{ inputs.provider }}` to env block, replaced `"${{ inputs.provider }}"` with `"$PROVIDER_INPUT"` in the shell condition.
   - outrider.yml 'Mint Remyx bot token' step: added `GITHUB_REPOSITORY_VALUE: ${{ github.repository }}` to env block, replaced `${{ github.repository }}` with `$GITHUB_REPOSITORY_VALUE`; replaced `${{ secrets.REMYX_API_KEY }}` with `$REMYX_API_KEY` (already available from job-level env).

3. github-env-injection: In outrider.yml 'Configure provider auth' step, added sanitization: `safe_model=$(printf '%s' "$MODEL_INPUT" | tr -d '\n\r')` and write `$safe_model` to GITHUB_ENV instead of raw `$MODEL_INPUT`.

