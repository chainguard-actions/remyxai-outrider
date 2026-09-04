<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.54

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.54** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags rather than immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the upstream tag is moved.

In action.yml:
- `uses: actions/setup-python@v5` (tag ref)
- `uses: actions/setup-node@v4` (tag ref)

In .github/workflows/outrider-daily.yml:
- `uses: actions/checkout@v4` (tag ref)

In .github/workflows/outrider.yml:
- `uses: actions/checkout@v4` (tag ref)

All should be pinned to full 40-character SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:432`
- `action.yml:437`
- `.github/workflows/outrider-daily.yml:43`
- `.github/workflows/outrider.yml:68`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands (rule a), allowing template substitution before the shell ever sees the value.

**action.yml — "Install gh-graph" step:**
```
install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph
```

**action.yml — "Recommend + implement + open PR" step:**
```
python ${{ github.action_path }}/src/run.py
```

**outrider.yml — "Configure provider auth" step** (attacker-controlled via `workflow_dispatch`):
```
if [ "${{ inputs.provider }}" = "zai" ]; then
```
`inputs.provider` is a `workflow_dispatch` input and is attacker-controllable. It is interpolated directly into the shell string rather than being read from an env var.

**outrider-weekly-refine.yml — "Pick candidate" step** (attacker-controlled via `workflow_dispatch`):
```
LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"
OVERRIDE="${{ inputs.pick-override || '' }}"
OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"
```
All three `inputs.*` values are `workflow_dispatch` inputs and are attacker-controllable. They are interpolated directly into the shell script rather than being passed through env vars.

Fix: move all `${{ ... }}` values into `env:` blocks and reference them as `$ENV_VAR` (double-quoted) inside the `run:` script.

Locations:

- `action.yml:492`
- `action.yml:591`
- `.github/workflows/outrider.yml:83`
- `.github/workflows/outrider-weekly-refine.yml:60`
- `.github/workflows/outrider-weekly-refine.yml:61`
- `.github/workflows/outrider-weekly-refine.yml:62`

### github-env-injection (severity: high)

Multiple `run:` blocks write inherited process environment variables to `$GITHUB_ENV` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A caller-controlled value containing newlines can inject arbitrary environment variable assignments into subsequent steps.

**action.yml — "Configure backend from provider input" step:**
The env vars `ZAI_API_KEY`, `MOONSHOT_API_KEY`, and `INPUT_MODEL` are inherited from the calling workflow's env block and written directly to `$GITHUB_ENV`:
```bash
echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"      # no sanitization
echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV" # no sanitization
echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"           # no sanitization
```

**outrider.yml — "Configure provider auth" step:**
The env vars `ZAI_API_KEY_SECRET`, `ANTHROPIC_API_KEY_SECRET`, and `MODEL_INPUT` (sourced from `${{ secrets.* }}` and `${{ inputs.model }}`) are written directly to `$GITHUB_ENV`:
```bash
echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV"  # no sanitization
echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV" # no sanitization
echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"               # no sanitization
```

Fix: sanitize each value before writing, e.g.:
```bash
safe=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r')
echo "ANTHROPIC_AUTH_TOKEN=$safe" >> "$GITHUB_ENV"
```

Locations:

- `action.yml:536`
- `action.yml:541`
- `action.yml:549`
- `action.yml:556`
- `action.yml:562`
- `action.yml:575`
- `.github/workflows/outrider.yml:84`
- `.github/workflows/outrider.yml:86`
- `.github/workflows/outrider.yml:89`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings:

1. **unpinned-uses**: Pinned actions/setup-python@v5→SHA a26af69, actions/setup-node@v4→SHA 49933ea, actions/checkout@v4→SHA 11d5960 (in both outrider-daily.yml and outrider.yml).

2. **script-injection**: Moved all ${{ }} expressions from run: blocks to env: blocks:
   - action.yml Install gh-graph: github.action_path → ACTION_PATH env var
   - action.yml Recommend step: github.action_path → ACTION_PATH env var
   - outrider.yml Configure provider auth: inputs.provider → PROVIDER_INPUT env var
   - outrider-weekly-refine.yml Pick candidate: inputs.lookback-days, inputs.pick-override, inputs.pick-override-arxiv → INPUT_LOOKBACK_DAYS, INPUT_OVERRIDE, INPUT_OVERRIDE_ARXIV env vars

3. **github-env-injection**: Added printf/tr sanitization before all GITHUB_ENV writes:
   - action.yml: ZAI_API_KEY, MOONSHOT_API_KEY, INPUT_MODEL all sanitized with `printf '%s' "$VAR" | tr -d '\n\r'`
   - outrider.yml: ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, MODEL_INPUT all sanitized

4. **static-unsanitized-env-write**: Covered by the INPUT_MODEL sanitization fix in action.yml.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 3 findings across 2 workflow files:

1. outrider.yml (script-injection): Moved `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` from the 'Mint Remyx bot token' run: shell string into the step's env: block as REMYX_API_KEY_SECRET and GITHUB_REPOSITORY_VALUE. The shell script now references these as plain environment variables.

2. outrider-weekly-refine.yml (github-env-injection, lines 131-132): Added `printf '%s' ... | tr -d '\n\r'` sanitization for both BRANCH and ARXIV values before writing them to $GITHUB_OUTPUT, preventing newline injection from workflow_dispatch user-controlled inputs.

3. outrider-weekly-refine.yml (github-env-injection, line 196): Replaced the static 'GAP_EOF' heredoc delimiter with a randomly generated delimiter using `openssl rand -hex 16`, preventing LLM-generated content from containing the delimiter string and injecting additional output variables into the $GITHUB_OUTPUT file.

