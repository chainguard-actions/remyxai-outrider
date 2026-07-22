<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.20** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the tag is moved.

In action.yml:
- `uses: actions/setup-python@v5` (tag, not SHA)
- `uses: actions/setup-node@v4` (tag, not SHA)

In .github/workflows/outrider.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

In .github/workflows/outrider-daily.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

All should be pinned to their full commit SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:476`
- `action.yml:480`
- `.github/workflows/outrider.yml:37`
- `.github/workflows/outrider-daily.yml:30`

### script-injection (severity: high)

Multiple `run:` blocks interpolate GitHub Actions expressions (`${{ ... }}`) directly into shell command strings. This causes YAML template substitution to occur before the shell parses the command, allowing an attacker to inject arbitrary shell metacharacters.

(a) `.github/workflows/outrider.yml` — "Configure provider auth" step: `${{ inputs.provider }}` is interpolated directly in the shell `if` condition:
  `if [ "${{ inputs.provider }}" = "zai" ]; then`
A workflow_dispatch caller can supply a value with shell metacharacters.

(a) `.github/workflows/outrider.yml` — "Mint Remyx bot token" step: `${{ github.repository }}` is interpolated directly in a curl `-d` argument:
  `-d "{\"repo\": \"${{ github.repository }}\"}"` 
github.repository flows through YAML template substitution before the shell sees it.

(a) `action.yml` — "Install gh-graph selection-pass tool on PATH" step: `${{ github.action_path }}` is interpolated directly in the shell command:
  `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`

(a) `action.yml` — "Recommend + implement + open PR" step: `${{ github.action_path }}` is interpolated directly in the shell command:
  `python ${{ github.action_path }}/src/run.py`

Fix: move all expressions into `env:` variables and reference them as `"$VAR"` in the shell script.

Locations:

- `.github/workflows/outrider.yml:51`
- `.github/workflows/outrider.yml:63`
- `action.yml:476`
- `action.yml:526`

### github-env-injection (severity: high)

The "Configure provider auth" step in `.github/workflows/outrider.yml` writes the value of `$MODEL_INPUT` to `$GITHUB_ENV` without sanitization. `MODEL_INPUT` is set from `${{ inputs.model }}` (a workflow_dispatch user-controlled input). An attacker can inject newlines into the model input to add arbitrary key=value pairs to the runner's environment for subsequent steps.

Offending code:
```
env:
  MODEL_INPUT: ${{ inputs.model }}
run: |
  if [ -n "$MODEL_INPUT" ]; then
    echo "ANTHROPIC_MODEL=$MODEL_INPUT" >> "$GITHUB_ENV"   # FAIL: no tr -d newlines
  fi
```

Fix: sanitize before writing:
```
  safe=$(printf '%s' "$MODEL_INPUT" | tr -d '\n\r')
  echo "ANTHROPIC_MODEL=$safe" >> "$GITHUB_ENV"
```

Locations:

- `.github/workflows/outrider.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings:

1. unpinned-uses: Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065, actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, and actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 in both workflow files.

2. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks — inputs.provider → PROVIDER_INPUT in outrider.yml Configure provider auth step; secrets.REMYX_API_KEY and github.repository → REMYX_API_KEY_SECRET and GITHUB_REPOSITORY_VALUE in Mint Remyx bot token step; github.action_path → ACTION_PATH in both action.yml steps (Install gh-graph and Recommend + implement + open PR).

3. github-env-injection: Sanitized MODEL_INPUT before writing to $GITHUB_ENV using printf '%s' "$MODEL_INPUT" | tr -d '\n\r' to strip newlines, preventing environment variable injection attacks.

