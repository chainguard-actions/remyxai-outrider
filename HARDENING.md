<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.43

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.43** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags rather than full 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or overwritten.

In action.yml:
- `uses: actions/setup-python@v5` (tag, not SHA)
- `uses: actions/setup-node@v4` (tag, not SHA)

In .github/workflows/outrider-daily.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

In .github/workflows/outrider.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

Locations:

- `action.yml:576`
- `action.yml:580`
- `.github/workflows/outrider-daily.yml:37`
- `.github/workflows/outrider.yml:57`

### script-injection (severity: high)

GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings (sub-rule a), allowing an attacker to inject arbitrary shell commands.

**outrider-weekly-refine.yml** — "Pick candidate" step: `workflow_dispatch` inputs are interpolated directly into shell variable assignments:
```
LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"
OVERRIDE="${{ inputs.pick-override || '' }}"
OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"
```
An attacker with dispatch access can inject shell metacharacters (e.g. `'; malicious_cmd; '`).

**outrider.yml** — "Configure provider auth" step: `${{ inputs.provider }}` is interpolated directly inside a shell `if` test:
```
if [ "${{ inputs.provider }}" = "zai" ]; then
```
This is a `workflow_dispatch` input and can be attacker-controlled.

**outrider.yml** — "Mint Remyx bot token" step: `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` are interpolated directly inside a `run:` block:
```
token="$(curl -sf -X POST -H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}" -d "{\"repo\": \"${{ github.repository }}\"}" ...)"
```

**action.yml** — "Install gh-graph" step and "Recommend + implement" step: `${{ github.action_path }}` is interpolated directly inside `run:` blocks:
```
install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph
python ${{ github.action_path }}/src/run.py
```

Locations:

- `.github/workflows/outrider-weekly-refine.yml:57`
- `.github/workflows/outrider-weekly-refine.yml:58`
- `.github/workflows/outrider-weekly-refine.yml:59`
- `.github/workflows/outrider.yml:63`
- `.github/workflows/outrider.yml:73`
- `action.yml:591`
- `action.yml:700`

### github-env-injection (severity: high)

Inherited process environment variables (set from workflow inputs or caller-supplied secrets) are written directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline character in any of these values would allow injection of arbitrary environment variables into subsequent steps.

**action.yml** — "Configure backend from provider input" step:
- `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `$ZAI_API_KEY` is an inherited env var from the calling workflow, unsanitized.
- `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — `$MOONSHOT_API_KEY` is an inherited env var, unsanitized.
- `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `$INPUT_MODEL` is set from `${{ inputs.model }}` (a caller-controlled input), unsanitized.

**outrider.yml** — "Configure provider auth" step:
- `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV"` — `$ZAI_API_KEY_SECRET` is set from `${{ secrets.ZAI_API_KEY }}` via env:, unsanitized.
- `echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV"` — `$ANTHROPIC_API_KEY_SECRET` is set from `${{ secrets.ANTHROPIC_API_KEY }}` via env:, unsanitized.
- `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"` — `$MODEL_INPUT` is set from `${{ inputs.model }}` (a workflow_dispatch input), unsanitized.

Correct pattern: `safe=$(printf '%s' "$VAR" | tr -d '\n\r'); echo "KEY=$safe" >> "$GITHUB_ENV"`

Locations:

- `action.yml:648`
- `action.yml:655`
- `action.yml:666`
- `.github/workflows/outrider.yml:64`
- `.github/workflows/outrider.yml:66`
- `.github/workflows/outrider.yml:69`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings across action.yml, outrider-daily.yml, outrider.yml, and outrider-weekly-refine.yml:

1. **unpinned-uses**: Pinned actions/setup-python@v5, actions/setup-node@v4, and actions/checkout@v4 (×2) to full 40-char SHAs.

2. **script-injection**: Moved all ${{ }} expressions from run: shell strings into env: blocks:
   - outrider-weekly-refine.yml: inputs.lookback-days, inputs.pick-override, inputs.pick-override-arxiv → env: with shell default fallbacks (${VAR:-default})
   - outrider.yml Configure step: inputs.provider → PROVIDER_INPUT env var
   - outrider.yml Mint token step: secrets.REMYX_API_KEY and github.repository → env: block
   - action.yml Install gh-graph step: github.action_path → ACTION_PATH env var
   - action.yml Recommend step: github.action_path → ACTION_PATH env var (also fixed duplicate env: block from a prior partial edit)

3. **github-env-injection** + **static-unsanitized-env-write**: Added printf/tr sanitization before all GITHUB_ENV writes:
   - action.yml Configure backend step: ZAI_API_KEY, MOONSHOT_API_KEY, INPUT_MODEL all sanitized with `safe=$(printf '%s' "$VAR" | tr -d '\n\r')`
   - outrider.yml Configure provider auth step: ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, MODEL_INPUT all sanitized similarly

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in `.github/workflows/outrider-weekly-refine.yml` at lines 156-157. The `$BRANCH` and `$ARXIV` values (derived from workflow_dispatch inputs via env vars OVERRIDE_INPUT and OVERRIDE_ARXIV_INPUT) are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before being written to $GITHUB_OUTPUT, preventing newline injection attacks that could poison subsequent steps consuming `steps.pick.outputs.picked` and `steps.pick.outputs.arxiv`.

