<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.34

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.34** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or hijacked.

In action.yml:
- `uses: actions/setup-python@v5`
- `uses: actions/setup-node@v4`

In .github/workflows/outrider-daily.yml:
- `uses: actions/checkout@v4`

In .github/workflows/outrider.yml:
- `uses: actions/checkout@v4`

All should be pinned to full 40-character SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:401`
- `action.yml:405`
- `.github/workflows/outrider-daily.yml:36`
- `.github/workflows/outrider.yml:57`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings, bypassing shell quoting and allowing an attacker to inject arbitrary shell commands.

**outrider.yml — "Configure provider auth" step (~line 68):**
```
if [ "${{ inputs.provider }}" = "zai" ]; then
```
`inputs.provider` is a `workflow_dispatch` input that flows through YAML template substitution before the shell sees it. An attacker who can trigger the workflow can inject shell metacharacters.

**outrider.yml — "Mint Remyx bot token" step (~line 76):**
```
-d "{\"repo\": \"${{ github.repository }}\"}" 
```
`github.repository` is interpolated directly into a curl argument. Although GitHub-controlled, it still flows through YAML template substitution and is a script-injection vector per the check rules.

**outrider-weekly-refine.yml — "Pick candidate" step (~lines 55-57):**
```
LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}"
OVERRIDE="${{ inputs.pick-override || '' }}"
OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}"
```
All three `workflow_dispatch` inputs are interpolated directly into shell variable assignments. An attacker who can trigger the workflow can inject shell metacharacters.

**action.yml — "Install gh-graph" and "Recommend + implement" steps:**
```
install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph
python ${{ github.action_path }}/src/run.py
```
`github.action_path` is interpolated directly into run blocks. All `${{ ... }}` expressions in run blocks are script-injection findings per the check rules.

Fix: move all expressions into `env:` variables and reference them as `"$VAR"` in the shell script.

Locations:

- `.github/workflows/outrider.yml:68`
- `.github/workflows/outrider.yml:76`
- `.github/workflows/outrider-weekly-refine.yml:55`
- `.github/workflows/outrider-weekly-refine.yml:56`
- `.github/workflows/outrider-weekly-refine.yml:57`
- `action.yml:430`
- `action.yml:601`

### github-env-injection (severity: high)

Unsanitized values derived from workflow-controlled (untrusted) sources are written to `$GITHUB_ENV` without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. A newline character in any of these values would allow an attacker to inject arbitrary environment variable definitions into subsequent steps.

**action.yml — "Configure backend from provider input" step:**
- `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `ZAI_API_KEY` is an inherited env var set by the calling workflow (workflow-controlled, treated as untrusted for composite actions).
- `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — `MOONSHOT_API_KEY` is an inherited env var set by the calling workflow.
- `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `INPUT_MODEL` is sourced from `${{ inputs.model }}` (an action input).

**outrider.yml — "Configure provider auth" step:**
- `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"` — `MODEL_INPUT` is sourced from `${{ inputs.model }}` (a workflow_dispatch input).

Fix: sanitize each value before writing, e.g.:
```bash
safe=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r')
echo "ANTHROPIC_AUTH_TOKEN=$safe" >> "$GITHUB_ENV"
```

Locations:

- `action.yml:530`
- `action.yml:540`
- `action.yml:560`
- `.github/workflows/outrider.yml:74`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:636`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all 4 findings:

1. unpinned-uses: Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065, actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, and both actions/checkout@v4 references to SHA 11d5960a326750d5838078e36cf38b85af677262.

2. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks in outrider.yml (inputs.provider → PROVIDER_INPUT, secrets.REMYX_API_KEY → REMYX_API_KEY_SECRET, github.repository → GITHUB_REPOSITORY_VALUE), outrider-weekly-refine.yml (three inputs → INPUT_LOOKBACK_DAYS/INPUT_OVERRIDE/INPUT_OVERRIDE_ARXIV), and action.yml (github.action_path → ACTION_PATH in both Install gh-graph and Recommend + implement steps).

3. github-env-injection: Added printf/tr sanitization before all writes to $GITHUB_ENV: ZAI_API_KEY (safe_zai), MOONSHOT_API_KEY (safe_moonshot), and INPUT_MODEL (safe_model) in action.yml; MODEL_INPUT (safe_model) in outrider.yml.

4. static-unsanitized-env-write: Same fix as github-env-injection — INPUT_MODEL in action.yml's Configure backend step is now sanitized before writing ANTHROPIC_MODEL to $GITHUB_ENV.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in .github/workflows/outrider-weekly-refine.yml at lines 148-149. Before writing BRANCH and ARXIV to $GITHUB_OUTPUT, both values are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` to strip embedded newlines/carriage-returns. This prevents a workflow_dispatch caller from injecting arbitrary key=value pairs into the step output context via the `inputs.pick-override` or `inputs.pick-override-arxiv` inputs.

