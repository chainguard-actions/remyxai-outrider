<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.40

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.40** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable version tags instead of immutable 40-character SHA digests. In action.yml: actions/setup-python@v5 and actions/setup-node@v4. In outrider-daily.yml: actions/checkout@v4. In outrider.yml: actions/checkout@v4.

Locations:

- `action.yml:248`
- `action.yml:252`
- `.github/workflows/outrider-daily.yml:36`
- `.github/workflows/outrider.yml:57`

### script-injection (severity: high)

Multiple run: blocks interpolate ${{ ... }} expressions directly into shell command strings (sub-rule a). (1) action.yml Install gh-graph step: install -m 0755 "${{ github.action_path }}/src/gh_graph.py" — github.action_path interpolated directly. (2) action.yml Recommend step: python ${{ github.action_path }}/src/run.py — github.action_path interpolated directly. (3) outrider.yml Configure provider auth step: if [ "${{ inputs.provider }}" = "zai" ] — attacker-controlled inputs.provider (workflow_dispatch) interpolated directly into shell if-condition. (4) outrider.yml Mint Remyx bot token step: -H "Authorization: Bearer ${{ secrets.REMYX_API_KEY }}" and -d "{\"repo\": \"${{ github.repository }}\"}" — expressions interpolated directly into curl command. (5) outrider-weekly-refine.yml Pick candidate step: LOOKBACK_DAYS="${{ inputs.lookback-days || '7' }}", OVERRIDE="${{ inputs.pick-override || '' }}", OVERRIDE_ARXIV="${{ inputs.pick-override-arxiv || '' }}" — all three attacker-controlled workflow_dispatch inputs interpolated directly into shell variable assignments.

Locations:

- `action.yml:270`
- `action.yml:399`
- `.github/workflows/outrider.yml:68`
- `.github/workflows/outrider.yml:76`
- `.github/workflows/outrider-weekly-refine.yml:52`
- `.github/workflows/outrider-weekly-refine.yml:53`
- `.github/workflows/outrider-weekly-refine.yml:54`

### github-env-injection (severity: high)

Multiple run: blocks write caller-controlled or inherited env var values to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). In action.yml Configure backend step: echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV" (ZAI_API_KEY is an inherited process env var set by the calling workflow), echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV" (same pattern for MOONSHOT_API_KEY), and echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV" (INPUT_MODEL comes from inputs.model via the step env: block) — all without sanitization. In outrider.yml Configure provider auth step: echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV", echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV", and echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV" (MODEL_INPUT from inputs.model) — all without sanitization. A newline in any value allows injecting arbitrary env vars into subsequent steps.

Locations:

- `action.yml:335`
- `action.yml:343`
- `action.yml:352`
- `.github/workflows/outrider.yml:69`
- `.github/workflows/outrider.yml:71`
- `.github/workflows/outrider.yml:74`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings across action.yml and the three workflow files:

1. unpinned-uses: Pinned actions/setup-python@v5, actions/setup-node@v4, and actions/checkout@v4 (×2) to full 40-char SHA digests with tag comments.

2. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks — github.action_path in two action.yml steps, inputs.provider in outrider.yml Configure step, secrets.REMYX_API_KEY and github.repository in outrider.yml Mint token step, and three inputs.* in outrider-weekly-refine.yml Pick candidate step.

3. github-env-injection: Added printf/tr sanitization before all GITHUB_ENV writes in action.yml Configure backend step (ZAI_API_KEY, MOONSHOT_API_KEY, INPUT_MODEL) and outrider.yml Configure provider auth step (ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, MODEL_INPUT).

4. static-unsanitized-env-write: Covered by the github-env-injection fix to the Configure backend step in action.yml.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/outrider-weekly-refine.yml. In the 'Pick candidate from past week's drafter output' step, both BRANCH (derived from inputs.pick-override via INPUT_OVERRIDE env var) and ARXIV (derived from inputs.pick-override-arxiv via INPUT_OVERRIDE_ARXIV env var) are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before being written to $GITHUB_OUTPUT. The sanitized values are stored in safe_branch and safe_arxiv variables respectively, which are then used in the echo statements.

