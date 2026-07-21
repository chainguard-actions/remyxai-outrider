<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.13** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses mutable tag refs instead of pinned SHA hashes for two actions: `actions/setup-python@v5` and `actions/setup-node@v4`. These should be pinned to full 40-character commit SHAs to prevent supply-chain attacks.

Locations:

- `action.yml:296`
- `action.yml:301`

### unpinned-uses (severity: high)

.github/workflows/outrider.yml uses a mutable tag ref `actions/checkout@v4` instead of a pinned SHA hash. This should be pinned to a full 40-character commit SHA to prevent supply-chain attacks.

Locations:

- `.github/workflows/outrider.yml:43`

### script-injection (severity: high)

Sub-rule (a): In the 'Configure provider auth' step, `${{ inputs.provider }}` is directly interpolated inside a run: shell command string: `if [ "${{ inputs.provider }}" = "zai" ]; then`. The `inputs.provider` value is a workflow_dispatch choice input that flows through YAML template substitution before the shell sees it, enabling script injection if the value contains shell metacharacters.

Locations:

- `.github/workflows/outrider.yml:56`

### script-injection (severity: high)

Sub-rule (a): In the 'Mint Remyx bot token' step, `${{ github.repository }}` is directly interpolated inside a run: shell command string inside a curl -d JSON argument: `-d "{\"repo\": \"${{ github.repository }}\"}"`. Any expression interpolated directly in a run: block is a script-injection risk regardless of context.

Locations:

- `.github/workflows/outrider.yml:68`

### script-injection (severity: high)

Sub-rule (a): In the 'Install gh-graph selection-pass tool on PATH' step, `${{ github.action_path }}` is directly interpolated inside a run: shell command string: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. Any ${{ }} expression inside a run: block is a script-injection finding.

Locations:

- `action.yml:316`

### script-injection (severity: high)

Sub-rule (a): In the 'Recommend + implement + open PR' step, `${{ github.action_path }}` is directly interpolated inside a run: shell command string: `python ${{ github.action_path }}/src/run.py`. Any ${{ }} expression inside a run: block is a script-injection finding.

Locations:

- `action.yml:393`

### github-env-injection (severity: high)

In the 'Configure provider auth' step, the env var `MODEL_INPUT` is set from `inputs.model` (an untrusted workflow_dispatch input) and then written to `$GITHUB_ENV` without sanitization: `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"`. An attacker-supplied newline in `inputs.model` could inject arbitrary environment variables. The required sanitization step (`safe=$(printf '%s' "$MODEL_INPUT" | tr -d '\n\r')`) is missing before the write.

Locations:

- `.github/workflows/outrider.yml:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 7 findings across action.yml and .github/workflows/outrider.yml:

1. Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5 (action.yml)
2. Pinned actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4 (action.yml)
3. Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4 (outrider.yml)
4. Fixed script injection: moved inputs.provider into env block as PROVIDER_INPUT (outrider.yml)
5. Fixed script injection: moved github.repository and secrets.REMYX_API_KEY into env block in Mint Remyx bot token step (outrider.yml)
6. Fixed script injection: moved github.action_path into env block as ACTION_PATH in gh-graph install step (action.yml)
7. Fixed script injection: moved github.action_path into existing env block as ACTION_PATH in recommend step (action.yml)
8. Fixed github-env-injection: added printf/tr sanitization for MODEL_INPUT before writing to GITHUB_ENV (outrider.yml)

