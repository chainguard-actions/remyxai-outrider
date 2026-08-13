<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.52

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.52** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved.

In action.yml:
- `uses: actions/setup-python@v5` (tag, not SHA)
- `uses: actions/setup-node@v4` (tag, not SHA)

In .github/workflows/outrider-daily.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

In .github/workflows/outrider.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

Locations:

- `action.yml:471`
- `action.yml:475`
- `.github/workflows/outrider-daily.yml:37`
- `.github/workflows/outrider.yml:68`

### script-injection (severity: high)

Multiple `run:` blocks interpolate `${{ ... }}` expressions directly into shell command strings (rule a), allowing an attacker who controls the expression value to inject arbitrary shell commands.

**action.yml — "Install gh-graph selection-pass tool on PATH" step:**
```
install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph
```
`github.action_path` is a GitHub context value interpolated directly into the shell command.

**action.yml — "Recommend + implement + open PR" step:**
```
python ${{ github.action_path }}/src/run.py
```
Same issue — `github.action_path` interpolated directly into the shell command without quoting or env-var indirection.

**outrider.yml — "Configure provider auth" step:**
```
if [ "${{ inputs.provider }}" = "zai" ]; then
```
`inputs.provider` is a `workflow_dispatch` input (attacker-controllable) interpolated directly into the shell `if` condition.

**outrider.yml — "Mint Remyx bot token" step:**
```
-H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}"
-d "{\"repo\": \"${{ github.repository }}\"}"
```
Both `secrets.REMYX_API_KEY` and `github.repository` are interpolated directly into the shell command string.

**outrider-weekly-refine.yml — "Pick candidate from past week's drafter output" step:**
```
LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"
OVERRIDE="${{ inputs.pick-override || '' }}"
OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"
```
All three `workflow_dispatch` inputs are interpolated directly into shell variable assignments. An attacker supplying a value like `'; malicious_command; echo '` via `workflow_dispatch` can execute arbitrary commands.

Locations:

- `action.yml:490`
- `action.yml:601`
- `.github/workflows/outrider.yml:82`
- `.github/workflows/outrider.yml:91`
- `.github/workflows/outrider-weekly-refine.yml:52`
- `.github/workflows/outrider-weekly-refine.yml:53`
- `.github/workflows/outrider-weekly-refine.yml:54`

### github-env-injection (severity: high)

Multiple `run:` blocks write inherited or input-derived environment variables to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`), allowing newline injection that can set arbitrary environment variables for subsequent steps.

**action.yml — "Configure backend from provider input" step:**
```bash
echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"      # ZAI_API_KEY inherited from calling workflow
echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV" # MOONSHOT_API_KEY inherited from calling workflow
echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"           # INPUT_MODEL = ${{ inputs.model }}
```
`ZAI_API_KEY` and `MOONSHOT_API_KEY` are inherited process env vars set by the calling workflow (untrusted per the composite-action rule). `INPUT_MODEL` is mapped from `inputs.model` in the step's `env:` block. None of these values are sanitized before being written to `$GITHUB_ENV`.

**outrider.yml — "Configure provider auth" step:**
```bash
echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV"  # from secrets.ZAI_API_KEY via env:
echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV" # from secrets.ANTHROPIC_API_KEY via env:
echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"               # MODEL_INPUT = ${{ inputs.model }}
```
`MODEL_INPUT` is sourced from `inputs.model` (a `workflow_dispatch` input). All three writes lack the `tr -d '\n\r'` sanitization step.

Locations:

- `action.yml:549`
- `action.yml:555`
- `action.yml:566`
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

Fixed all 4 findings across action.yml and .github/workflows/ files:

1. unpinned-uses: Pinned actions/setup-python@v5, actions/setup-node@v4, and actions/checkout@v4 (×2) to full 40-char commit SHAs with tag comments.

2. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks. Specifically: github.action_path in two action.yml steps, inputs.provider in outrider.yml Configure step, secrets.REMYX_API_KEY and github.repository in outrider.yml Mint step, and three workflow_dispatch inputs in outrider-weekly-refine.yml Pick step.

3. github-env-injection: Replaced bare echo writes to $GITHUB_ENV with printf '%s' "$VAR" | tr -d '\n\r' sanitization for ZAI_API_KEY, MOONSHOT_API_KEY, and INPUT_MODEL in action.yml, and for ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, and MODEL_INPUT in outrider.yml.

4. static-unsanitized-env-write: The INPUT_MODEL write to $GITHUB_ENV in action.yml was sanitized as part of the github-env-injection fix.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/outrider-weekly-refine.yml at line 130. The BRANCH variable (derived from user-controlled inputs.pick-override via INPUT_OVERRIDE env var) was written directly to $GITHUB_OUTPUT without sanitization. Added sanitization using `safe_branch=$(printf '%s' "$BRANCH" | tr -d '\n\r')` and `safe_arxiv=$(printf '%s' "$ARXIV" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT, preventing newline injection attacks that could poison subsequent step outputs.

