<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.36

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.36** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved:
- `uses: actions/setup-python@v5`
- `uses: actions/setup-node@v4`
These should be pinned to their full commit SHAs (e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`).

Locations:

- `action.yml:435`
- `action.yml:440`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks directly interpolate `${{ github.action_path }}` inside shell command strings. Any `${{ ... }}` expression interpolated directly into a `run:` script is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it, bypassing shell quoting.

Offending lines:
1. `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` (step: "Install gh-graph selection-pass tool on PATH")
2. `python ${{ github.action_path }}/src/run.py` (step: "Recommend + implement + open PR")

Fix: use the `$GITHUB_ACTION_PATH` environment variable instead, which is already available as a safe shell variable: `install -m 0755 "$GITHUB_ACTION_PATH/src/gh_graph.py" ...` and `python "$GITHUB_ACTION_PATH/src/run.py"`.

Locations:

- `action.yml:462`
- `action.yml:590`

### github-env-injection (severity: high)

The "Configure backend from provider input" step writes inherited process env vars directly to $GITHUB_ENV without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). These env vars are set by the calling workflow's `env:` block and are therefore workflow-controlled (untrusted). A newline character embedded in any of these values would allow injection of additional KEY=VALUE pairs into GITHUB_ENV, potentially overwriting sensitive variables for subsequent steps.

Offending writes:
1. `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — $ZAI_API_KEY is an inherited env var from the calling workflow
2. `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — $MOONSHOT_API_KEY is an inherited env var from the calling workflow
3. `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — $INPUT_MODEL is derived from `${{ inputs.model }}` (an action input)

Fix example:
```bash
safe=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r')
echo "ANTHROPIC_AUTH_TOKEN=$safe" >> "$GITHUB_ENV"
```

Locations:

- `action.yml:519`
- `action.yml:527`
- `action.yml:541`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:636`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings in hardened/action/action.yml:
1. unpinned-uses: Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5 and actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4.
2. script-injection: Replaced both `${{ github.action_path }}` expressions in run: blocks with the safe `$GITHUB_ACTION_PATH` shell variable (in the gh-graph install step and the recommend step).
3. github-env-injection (ZAI_API_KEY): Added `safe_zai=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r')` before writing ANTHROPIC_AUTH_TOKEN to $GITHUB_ENV.
4. github-env-injection (MOONSHOT_API_KEY): Added `safe_moonshot=$(printf '%s' "$MOONSHOT_API_KEY" | tr -d '\n\r')` before writing ANTHROPIC_AUTH_TOKEN to $GITHUB_ENV.
5. static-unsanitized-env-write / github-env-injection (INPUT_MODEL): Added `safe_model=$(printf '%s' "$INPUT_MODEL" | tr -d '\n\r')` before writing ANTHROPIC_MODEL to $GITHUB_ENV.

