<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.49

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.49** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses mutable tag-pinned action references instead of immutable SHA-pinned refs. Failing references: `uses: actions/setup-python@v5` and `uses: actions/setup-node@v4`. These should be pinned to full 40-character commit SHAs to prevent supply-chain attacks.

Locations:

- `action.yml:453`
- `action.yml:458`

### unpinned-uses (severity: high)

examples/workflows/with-cocoindex.yml uses mutable tag-pinned action references instead of immutable SHA-pinned refs. Failing references: `uses: actions/checkout@v4` and `uses: remyxai/outrider@v1`. These should be pinned to full 40-character commit SHAs to prevent supply-chain attacks.

Locations:

- `examples/workflows/with-cocoindex.yml:31`
- `examples/workflows/with-cocoindex.yml:57`

### script-injection (severity: high)

Sub-rule (a): `${{ github.action_path }}` is interpolated directly inside a `run:` shell command string in the 'Install gh-graph selection-pass tool on PATH' step. The offending line is: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection risk as it flows through YAML template substitution before the shell processes it.

Locations:

- `action.yml:480`

### script-injection (severity: high)

Sub-rule (a): `${{ github.action_path }}` is interpolated directly inside a `run:` shell command string in the 'Recommend + implement + open PR' step. The offending line is: `python ${{ github.action_path }}/src/run.py`. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection risk as it flows through YAML template substitution before the shell processes it.

Locations:

- `action.yml:660`

### github-env-injection (severity: high)

In the 'Configure backend from provider input' step, inherited process env vars sourced from workflow-controlled inputs are written to $GITHUB_ENV without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Specifically: (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — ZAI_API_KEY is set by the calling workflow's env block; (2) `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — MOONSHOT_API_KEY is set by the calling workflow's env block; (3) `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — INPUT_MODEL is derived from `${{ inputs.model }}` in the step's env block. All three writes allow newline injection into GITHUB_ENV, enabling an attacker to inject arbitrary environment variables.

Locations:

- `action.yml:560`
- `action.yml:567`
- `action.yml:580`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all 6 findings:
1. Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5 in action.yml
2. Pinned actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4 in action.yml
3. Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4 in examples/workflows/with-cocoindex.yml
4. Pinned remyxai/outrider@v1 → @f6b77e674dc47c2283ba4f269938bf6794738ee0 # v1 in examples/workflows/with-cocoindex.yml
5. Fixed script-injection in 'Install gh-graph' step: moved ${{ github.action_path }} to env: block as ACTION_PATH
6. Fixed script-injection in 'Recommend + implement + open PR' step: moved ${{ github.action_path }} to env: block as ACTION_PATH
7. Fixed github-env-injection: sanitized ZAI_API_KEY write with printf/tr before writing to $GITHUB_ENV
8. Fixed github-env-injection: sanitized MOONSHOT_API_KEY write with printf/tr before writing to $GITHUB_ENV
9. Fixed static-unsanitized-env-write: sanitized INPUT_MODEL write with printf/tr before writing to $GITHUB_ENV

### Iteration 2

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 6 findings across 3 workflow files:

1. outrider-daily.yml: Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4

2. outrider.yml:
   - Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4
   - Configure provider auth step: moved ${{ inputs.provider }} into env block as PROVIDER_INPUT; sanitized all GITHUB_ENV writes with printf | tr -d '\n\r'
   - Mint Remyx bot token step: moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} into env block as REMYX_API_KEY_SECRET and GITHUB_REPO

3. outrider-weekly-refine.yml:
   - Pick candidate step: moved ${{ inputs.lookback-days }}, ${{ inputs.pick-override }}, ${{ inputs.pick-override-arxiv }} into env block as INPUT_LOOKBACK_DAYS, INPUT_OVERRIDE, INPUT_OVERRIDE_ARXIV
   - Sanitized BRANCH and ARXIV with printf | tr -d '\n\r' before writing to GITHUB_OUTPUT

