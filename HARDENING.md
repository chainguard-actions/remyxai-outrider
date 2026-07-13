<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.7.20** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple action references use mutable version tags instead of pinned full-length SHA digests, making the action vulnerable to supply-chain attacks if the tag is moved. Failing references: action.yml uses actions/setup-python@v5 and actions/setup-node@v4; .github/workflows/outrider.yml uses actions/checkout@v4; .github/workflows/outrider-daily.yml uses actions/checkout@v4.

Locations:

- `action.yml:250`
- `action.yml:254`
- `.github/workflows/outrider.yml:41`
- `.github/workflows/outrider-daily.yml:31`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings, bypassing shell quoting. (1) .github/workflows/outrider.yml line 54: `if [ "${{ inputs.provider }}" = "zai" ]` — inputs.provider is a workflow_dispatch input interpolated directly into the shell command. (2) .github/workflows/outrider.yml line 66: `${{ github.repository }}` is interpolated directly into a curl -d JSON body inside a run: block. (3) action.yml: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py"` and `python ${{ github.action_path }}/src/run.py` — github.action_path (a github.* context) is interpolated directly into run: shell commands.

Locations:

- `.github/workflows/outrider.yml:54`
- `.github/workflows/outrider.yml:66`
- `action.yml:285`
- `action.yml:380`

### github-env-injection (severity: high)

In .github/workflows/outrider.yml, the env var MODEL_INPUT is set from ${{ inputs.model }} (line 52) and then written unsanitized to $GITHUB_ENV via `echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"` (line 60). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write. An attacker-controlled newline in inputs.model could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/outrider.yml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings: (1) Pinned 4 action references to full commit SHAs: actions/setup-python@v5→a26af69..., actions/setup-node@v4→49933ea..., actions/checkout@v4→34e1148... (used in both outrider.yml and outrider-daily.yml). (2) Fixed script injection in 4 locations by moving ${{ }} expressions into env: blocks: inputs.provider→PROVIDER_INPUT, github.repository→GITHUB_REPOSITORY_VALUE, github.action_path→ACTION_PATH (in both gh-graph install step and recommend step). Also moved secrets.REMYX_API_KEY to env: in the Mint step. (3) Fixed github-env-injection by sanitizing MODEL_INPUT with printf '%s' "$MODEL_INPUT" | tr -d '\n\r' before writing ANTHROPIC_MODEL to $GITHUB_ENV.

