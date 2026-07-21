<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.36

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.36** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised.

Failing references:
- action.yml: `uses: actions/setup-python@v5`
- action.yml: `uses: actions/setup-node@v4`
- .github/workflows/outrider-daily.yml: `uses: actions/checkout@v4`
- .github/workflows/outrider.yml: `uses: actions/checkout@v4`

Locations:

- `action.yml:393`
- `action.yml:397`
- `.github/workflows/outrider-daily.yml:38`
- `.github/workflows/outrider.yml:57`

### script-injection (severity: high)

Multiple `run:` blocks interpolate `${{ ... }}` expressions directly into shell command strings (sub-rule a), bypassing shell quoting and enabling command injection.

**action.yml — Install gh-graph step:**
`install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`

**action.yml — Recommend + implement step:**
`python ${{ github.action_path }}/src/run.py`

**outrider.yml — Configure provider auth step:**
`if [ "${{ inputs.provider }}" = "zai" ]; then` — `inputs.provider` is a workflow_dispatch input directly interpolated into a shell conditional, enabling injection of shell metacharacters.

**outrider.yml — Mint Remyx bot token step:**
`-H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}"` and `-d "{\"repo\": \"${{ github.repository }}\"}"` are interpolated directly into a shell command string.

**outrider-weekly-refine.yml — Pick candidate step:**
`LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"`
`OVERRIDE="${{ inputs.pick-override || '' }}"`
`OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"`
All three `workflow_dispatch` inputs are interpolated directly into shell variable assignments without going through an `env:` block first.

Locations:

- `action.yml:419`
- `action.yml:530`
- `.github/workflows/outrider.yml:67`
- `.github/workflows/outrider.yml:78`
- `.github/workflows/outrider-weekly-refine.yml:55`
- `.github/workflows/outrider-weekly-refine.yml:56`
- `.github/workflows/outrider-weekly-refine.yml:57`

### github-env-injection (severity: high)

Multiple `run:` blocks write inherited process environment variables (set by the calling workflow) to `$GITHUB_ENV` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A newline in any of these values allows an attacker to inject arbitrary environment variable definitions into subsequent steps.

**action.yml — Configure backend from provider input step:**
- `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `$ZAI_API_KEY` is an inherited env var from the calling workflow, unsanitized.
- `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — `$MOONSHOT_API_KEY` is an inherited env var, unsanitized.
- `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `$INPUT_MODEL` is sourced from `inputs.model` via env:, unsanitized.

**outrider.yml — Configure provider auth step:**
- `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV"` — sourced from `secrets.ZAI_API_KEY` via env:, unsanitized.
- `echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV"` — sourced from `secrets.ANTHROPIC_API_KEY` via env:, unsanitized.
- `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"` — sourced from `inputs.model` via env:, unsanitized.

Locations:

- `action.yml:476`
- `action.yml:484`
- `action.yml:494`
- `.github/workflows/outrider.yml:68`
- `.github/workflows/outrider.yml:70`
- `.github/workflows/outrider.yml:73`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:636`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings across action.yml and three workflow files:

1. unpinned-uses: Pinned actions/setup-python@v5→SHA, actions/setup-node@v4→SHA, and actions/checkout@v4→SHA (in both outrider.yml and outrider-daily.yml) using full 40-char commit SHAs with tag comments.

2. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks. Fixed: gh-graph install step (github.action_path→ACTION_PATH), recommend step (github.action_path→ACTION_PATH), outrider.yml Configure provider auth step (inputs.provider→PROVIDER_INPUT), outrider.yml Mint Remyx bot token step (secrets.REMYX_API_KEY→REMYX_API_KEY_SECRET, github.repository→GITHUB_REPOSITORY_VALUE), outrider-weekly-refine.yml Pick candidate step (all three inputs→env vars).

3. github-env-injection + static-unsanitized-env-write: Added printf '%s' "$VAR" | tr -d '\n\r' sanitization before all writes to $GITHUB_ENV in action.yml Configure backend step (ZAI_API_KEY, MOONSHOT_API_KEY, INPUT_MODEL) and outrider.yml Configure provider auth step (ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, MODEL_INPUT).

Also fixed an accidental duplicate env: block in action.yml's recommend step that was introduced during editing.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two high-severity findings:

1. github-env-injection in outrider-weekly-refine.yml: Added printf/tr sanitization before writing $BRANCH and $ARXIV to $GITHUB_OUTPUT. Both values are now stripped of newline characters via `printf '%s' "$VAR" | tr -d '\n\r'` before being written, preventing newline injection attacks that could poison downstream step outputs.

2. script-injection in outrider.yml: Replaced the unsafe shell-interpolated curl JSON body (`-d "{\"repo\": \"$GITHUB_REPOSITORY_VALUE\"}"`) with a jq-constructed payload (`jq -n --arg repo "$GITHUB_REPOSITORY_VALUE" '{"repo": $repo}'`). This ensures the github.repository value is properly JSON-escaped before being sent in the HTTP request body, preventing JSON structure breakage or shell command injection.

