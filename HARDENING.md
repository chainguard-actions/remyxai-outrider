<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.53

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.53** was hardened automatically. 7 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell commands. Although `github.action_path` is not attacker-controlled in the same way as `inputs.*`, any `${{ ... }}` expression inside a `run:` block is a script-injection finding per the check rules. (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the 'Install gh-graph selection-pass tool on PATH' step. (2) `python ${{ github.action_path }}/src/run.py` in the 'Recommend + implement + open PR' step. Both should use the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:380`
- `action.yml:530`

### script-injection (severity: high)

Sub-rule (a): In .github/workflows/outrider.yml, the 'Configure provider auth' step directly interpolates the workflow_dispatch input `${{ inputs.provider }}` inside a `run:` shell command: `if [ "${{ inputs.provider }}" = "zai" ]; then`. A user-supplied value is template-expanded into the shell script before the shell parses it, enabling command injection. The value should be passed via an `env:` variable and referenced as `"$INPUT_PROVIDER"` instead.

Locations:

- `.github/workflows/outrider.yml:89`

### github-env-injection (severity: high)

In action.yml's 'Configure backend from provider input' step, the env var `INPUT_MODEL` (sourced from `${{ inputs.model }}`, a caller-controlled composite-action input) is written to `$GITHUB_ENV` without sanitization: `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"`. A value containing a newline can inject arbitrary environment variables into subsequent steps. The fix is to sanitize before writing: `safe=$(printf '%s' "$INPUT_MODEL" | tr -d '\n\r'); echo "ANTHROPIC_MODEL=$safe" >> "$GITHUB_ENV"`.

Locations:

- `action.yml:502`

### github-env-injection (severity: high)

In .github/workflows/outrider.yml's 'Configure provider auth' step, the env var `MODEL_INPUT` (sourced from `${{ inputs.model }}`, a workflow_dispatch input) is written to `$GITHUB_ENV` without sanitization: `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"`. A value containing a newline can inject arbitrary environment variables into subsequent steps. The fix is to sanitize before writing: `safe=$(printf '%s' "$MODEL_INPUT" | tr -d '\n\r'); echo "ANTHROPIC_MODEL=$safe" >> "$GITHUB_ENV"`.

Locations:

- `.github/workflows/outrider.yml:95`

### unpinned-uses (severity: high)

action.yml references two actions by mutable version tags instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved: (1) `uses: actions/setup-python@v5` — should be pinned to a SHA such as `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`. (2) `uses: actions/setup-node@v4` — should be pinned to a SHA such as `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`.

Locations:

- `action.yml:360`
- `action.yml:362`

### unpinned-uses (severity: high)

Workflow files reference actions by mutable version tags instead of full 40-character commit SHAs. (1) .github/workflows/outrider.yml: `uses: actions/checkout@v4`. (2) .github/workflows/outrider-daily.yml: `uses: actions/checkout@v4`. Both should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/outrider.yml:76`
- `.github/workflows/outrider-daily.yml:35`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, static-unsanitized-env-write

**Notes:**

Fixed all 7 findings across action.yml and .github/workflows/outrider.yml and .github/workflows/outrider-daily.yml:

1. script-injection (action.yml): Replaced two `${{ github.action_path }}` expressions in `run:` blocks with `$GITHUB_ACTION_PATH` environment variable.
2. script-injection (outrider.yml): Moved `${{ inputs.provider }}` from the `run:` block into the `env:` block as `INPUT_PROVIDER`, and updated the shell script to use `$INPUT_PROVIDER`.
3. github-env-injection (action.yml): Added `safe_model=$(printf '%s' "$INPUT_MODEL" | tr -d '\n\r')` sanitization before writing `ANTHROPIC_MODEL` to `$GITHUB_ENV`.
4. github-env-injection (outrider.yml): Added `safe_model=$(printf '%s' "$MODEL_INPUT" | tr -d '\n\r')` sanitization before writing `ANTHROPIC_MODEL` to `$GITHUB_ENV`.
5. unpinned-uses (action.yml): Pinned `actions/setup-python@v5` → SHA `a26af69be951a213d495a4c3e4e4022e16d87065` and `actions/setup-node@v4` → SHA `49933ea5288caeca8642d1e84afbd3f7d6820020`.
6. unpinned-uses (outrider.yml + outrider-daily.yml): Pinned `actions/checkout@v4` → SHA `11d5960a326750d5838078e36cf38b85af677262` in both workflow files.
7. static-unsanitized-env-write: Covered by the github-env-injection fix in action.yml.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three security findings: (1) outrider-weekly-refine.yml line 57: moved inputs.lookback-days, inputs.pick-override, and inputs.pick-override-arxiv from inline ${{ }} expressions in run: to an env: block, referencing them as $INPUT_LOOKBACK_DAYS, $INPUT_PICK_OVERRIDE, $INPUT_PICK_OVERRIDE_ARXIV in the shell script; (2) outrider.yml line 100: moved secrets.REMYX_API_KEY and github.repository from inline ${{ }} expressions in the Mint Remyx bot token run: block to an env: block as REMYX_API_KEY_SECRET and GITHUB_REPOSITORY_VALUE; (3) action.yml line 530: added printf '%s' | tr -d '\n\r' sanitization for ZAI_API_KEY and MOONSHOT_API_KEY before writing ANTHROPIC_AUTH_TOKEN to $GITHUB_ENV in the Configure backend from provider input step.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in hardened/action/.github/workflows/outrider-weekly-refine.yml. In the 'Pick candidate from past week's drafter output' step, BRANCH and ARXIV values (derived from user-controlled inputs pick-override and pick-override-arxiv) were written to $GITHUB_OUTPUT without sanitization. Added sanitization using `printf '%s' "$VAR" | tr -d '\n\r'` for both values, storing them in SAFE_BRANCH and SAFE_ARXIV before writing to $GITHUB_OUTPUT. This prevents newline injection attacks where a malicious caller could inject arbitrary key=value pairs into $GITHUB_OUTPUT.

### Iteration 4

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Configure provider auth' step in .github/workflows/outrider.yml. Both $ZAI_API_KEY_SECRET (written as ANTHROPIC_AUTH_TOKEN) and $ANTHROPIC_API_KEY_SECRET (written as ANTHROPIC_API_KEY) are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before being written to $GITHUB_ENV, preventing embedded newlines from injecting arbitrary environment variable assignments into subsequent steps. This matches the sanitization pattern already applied to MODEL_INPUT in the same step.

