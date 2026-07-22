<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.39

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.39** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two run: blocks in action.yml directly interpolate ${{ github.action_path }} inside shell command strings. Any ${{ ... }} expression inside a run: block is a script-injection risk regardless of context. (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the 'Install gh-graph selection-pass tool on PATH' step. (2) `python ${{ github.action_path }}/src/run.py` in the 'Recommend + implement + open PR' step. These should use the $GITHUB_ACTION_PATH environment variable instead.

Locations:

- `action.yml:345`
- `action.yml:530`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes inherited process env vars directly to $GITHUB_ENV without the required sanitization (printf '%s' ... | tr -d '\n\r'). This is a case (e) violation — composite actions inherit env from the calling workflow, so ZAI_API_KEY, MOONSHOT_API_KEY, and INPUT_MODEL (set from ${{ inputs.model }}) are workflow-controlled and must be sanitized before being written to $GITHUB_ENV. Offending lines: (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"`, (2) `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"`, (3) `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"`. A newline injected into any of these values could define additional environment variables in subsequent steps.

Locations:

- `action.yml:452`
- `action.yml:462`
- `action.yml:480`

### unpinned-uses (severity: high)

Two uses: references in action.yml use mutable version tags instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks. Failing references: (1) `uses: actions/setup-python@v5` — should be pinned to a full SHA. (2) `uses: actions/setup-node@v4` — should be pinned to a full SHA.

Locations:

- `action.yml:316`
- `action.yml:321`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings in hardened/action/action.yml:
1. unpinned-uses (lines 316, 321): Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5 and actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4.
2. script-injection (lines 345, 530): Replaced both ${{ github.action_path }} expressions inside run: blocks with the $GITHUB_ACTION_PATH environment variable.
3. github-env-injection (lines 452, 462, 480): Added printf '%s' ... | tr -d '\n\r' sanitization for ZAI_API_KEY, MOONSHOT_API_KEY, and INPUT_MODEL before writing to $GITHUB_ENV.
4. static-unsanitized-env-write (line 665): Same INPUT_MODEL sanitization fix covers this finding as well.

### Iteration 2

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 5 findings across 3 workflow files:

1. outrider-daily.yml: Pinned actions/checkout@v4 to full SHA 11d5960a326750d5838078e36cf38b85af677262.

2. outrider.yml: (a) Pinned actions/checkout@v4 to full SHA. (b) Fixed script-injection in 'Configure provider auth' by moving ${{ inputs.provider }} into env block as PROVIDER_INPUT. (c) Fixed script-injection in 'Mint Remyx bot token' by moving ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} into env block as REMYX_API_KEY_SECRET and GITHUB_REPO. (d) Fixed github-env-injection by sanitizing ZAI_API_KEY_SECRET, ANTHROPIC_API_KEY_SECRET, and MODEL_INPUT with printf '%s' ... | tr -d '\n\r' before writing to $GITHUB_ENV.

3. outrider-weekly-refine.yml: Fixed script-injection in 'Pick candidate' step by moving ${{ inputs.lookback-days || '7' }}, ${{ inputs.pick-override || '' }}, and ${{ inputs.pick-override-arxiv || '' }} into the step's env block as INPUT_LOOKBACK_DAYS, INPUT_PICK_OVERRIDE, and INPUT_PICK_OVERRIDE_ARXIV, then referencing them as plain shell variables.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in outrider-weekly-refine.yml at lines 130-131. The BRANCH and ARXIV variables (derived from workflow_dispatch inputs pick-override and pick-override-arxiv) are now sanitized with `printf '%s' ... | tr -d '\n\r'` before being written to $GITHUB_OUTPUT, preventing newline injection attacks that could poison subsequent steps consuming steps.pick.outputs.picked and steps.pick.outputs.arxiv.

