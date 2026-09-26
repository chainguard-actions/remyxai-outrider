<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.52

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.52** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml pin to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved:
- `uses: actions/setup-python@v5` (tag `v5`)
- `uses: actions/setup-node@v4` (tag `v4`)
These should be pinned to their full SHA digests, e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `action.yml:453`
- `action.yml:458`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks directly interpolate `${{ github.action_path }}` as a `${{ ... }}` expression inside the shell command string. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it, bypassing shell quoting.

1. In the "Install gh-graph selection-pass tool on PATH" step:
   `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`

2. In the "Recommend + implement + open PR" step:
   `python ${{ github.action_path }}/src/run.py`

The safe pattern is to use the `$GITHUB_ACTION_PATH` environment variable instead: `python "$GITHUB_ACTION_PATH/src/run.py"`.

Locations:

- `action.yml:470`
- `action.yml:560`

### github-env-injection (severity: high)

The "Configure backend from provider input" step writes workflow-controlled (caller-supplied) environment variables directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). These inherited process env vars are set by the calling workflow's `env:` block and could contain embedded newlines, allowing an attacker to inject additional key=value pairs into the runner environment:

1. `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `$ZAI_API_KEY` is caller-supplied and unsanitized.
2. `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — `$MOONSHOT_API_KEY` is caller-supplied and unsanitized.
3. `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `$INPUT_MODEL` comes from `${{ inputs.model }}` via env var and is unsanitized.

The fix is to sanitize each value before writing: `safe=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r'); echo "ANTHROPIC_AUTH_TOKEN=$safe" >> "$GITHUB_ENV"`.

Locations:

- `action.yml:519`
- `action.yml:527`
- `action.yml:537`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings in hardened/action/action.yml:
1. unpinned-uses: Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065 and actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, preserving the tag as a comment.
2. script-injection: Replaced both ${{ github.action_path }} expressions in run: blocks with $GITHUB_ACTION_PATH (the safe environment variable equivalent).
3. github-env-injection: Added sanitization (printf '%s' "$VAR" | tr -d '\n\r') before writing ZAI_API_KEY and MOONSHOT_API_KEY to $GITHUB_ENV as ANTHROPIC_AUTH_TOKEN.
4. static-unsanitized-env-write: Added sanitization before writing INPUT_MODEL to $GITHUB_ENV as ANTHROPIC_MODEL (same fix covers both github-env-injection and static-unsanitized-env-write findings for this variable).

