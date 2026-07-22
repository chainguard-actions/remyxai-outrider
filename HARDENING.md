<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.15** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses mutable version tags instead of pinned SHA digests for two actions: `actions/setup-python@v5` and `actions/setup-node@v4`. These tags can be moved to point to different (potentially malicious) commits without notice, creating a supply-chain risk.

Locations:

- `action.yml:278`
- `action.yml:282`

### unpinned-uses (severity: high)

.github/workflows/outrider.yml uses a mutable version tag instead of a pinned SHA digest: `actions/checkout@v4`. This tag can be moved to point to a different commit without notice.

Locations:

- `.github/workflows/outrider.yml:39`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` into shell commands before the shell processes them. (1) In the 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) In the 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py`. Any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk per the check rules.

Locations:

- `action.yml:290`
- `action.yml:358`

### script-injection (severity: high)

Sub-rule (a): The 'Configure provider auth' step in outrider.yml interpolates `${{ inputs.provider }}` (a workflow_dispatch user-controlled input) directly into a `run:` shell command: `if [ "${{ inputs.provider }}" = "zai" ]; then`. The 'Mint Remyx bot token' step also interpolates `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` directly into the `run:` block inside a curl command. All are direct expression interpolations inside `run:` blocks (sub-rule a).

Locations:

- `.github/workflows/outrider.yml:47`
- `.github/workflows/outrider.yml:57`

### github-env-injection (severity: high)

The 'Configure provider auth' step in outrider.yml writes workflow-controlled values to $GITHUB_ENV without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). Three unsanitized writes: (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY_SECRET" >> "$GITHUB_ENV"` where ZAI_API_KEY_SECRET comes from `${{ secrets.ZAI_API_KEY }}`; (2) `echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY_SECRET" >> "$GITHUB_ENV"` where ANTHROPIC_API_KEY_SECRET comes from `${{ secrets.ANTHROPIC_API_KEY }}`; (3) `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"` where MODEL_INPUT comes from `${{ inputs.model }}` (a user-controlled workflow_dispatch input). A value containing a newline can inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/outrider.yml:50`
- `.github/workflows/outrider.yml:52`
- `.github/workflows/outrider.yml:54`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 5 findings across action.yml and .github/workflows/outrider.yml:

1. action.yml: Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 and actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020.

2. outrider.yml: Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262.

3. action.yml script-injection: Moved ${{ github.action_path }} into env: blocks as ACTION_PATH in both the 'Install gh-graph' step and the 'Recommend + implement + open PR' step.

4. outrider.yml script-injection: Moved ${{ inputs.provider }} into env: as PROVIDER_INPUT in 'Configure provider auth'; moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} into env: vars in 'Mint Remyx bot token'.

5. outrider.yml github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization for all three GITHUB_ENV writes (ZAI_API_KEY → ANTHROPIC_AUTH_TOKEN, ANTHROPIC_API_KEY → ANTHROPIC_API_KEY, MODEL_INPUT → ANTHROPIC_MODEL).

