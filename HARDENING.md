<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.42

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.42** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two run: blocks in action.yml directly interpolate ${{ github.action_path }} (a github.* context expression) inside shell command strings. Per the script-injection check, ANY ${{ ... }} expression inside a run: block is a violation (rule a), regardless of which context it reads from.

1. "Install gh-graph selection-pass tool on PATH" step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`
2. "Recommend + implement + open PR" step: `python ${{ github.action_path }}/src/run.py`

Fix: use the `$GITHUB_ACTION_PATH` environment variable instead (e.g. `install -m 0755 "$GITHUB_ACTION_PATH/src/gh_graph.py"` and `python "$GITHUB_ACTION_PATH/src/run.py"`).

Locations:

- `action.yml:430`
- `action.yml:556`

### github-env-injection (severity: high)

The "Configure backend from provider input" step writes inherited process environment variables directly to $GITHUB_ENV without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). These variables are set by the calling workflow's env: block and are therefore workflow-controlled (untrusted) inputs:

1. `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — $ZAI_API_KEY is an inherited env var from the caller
2. `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — $MOONSHOT_API_KEY is an inherited env var from the caller
3. `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — $INPUT_MODEL is set from ${{ inputs.model }} in the step's env: block

A newline in any of these values could inject additional key=value pairs into GITHUB_ENV, allowing environment variable poisoning for subsequent steps. Fix: sanitize each value before writing, e.g. `safe=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r'); echo "ANTHROPIC_AUTH_TOKEN=$safe" >> "$GITHUB_ENV"`.

Locations:

- `action.yml:490`
- `action.yml:497`
- `action.yml:540`

### unpinned-uses (severity: high)

Two composite action steps use mutable version tags instead of pinned full-length SHA digests, making the action vulnerable to supply-chain attacks if the upstream action tags are moved or compromised.

- `uses: actions/setup-python@v5` — should be pinned to a full 40-character commit SHA
- `uses: actions/setup-node@v4` — should be pinned to a full 40-character commit SHA

Fix: replace each tag with the corresponding full SHA, e.g. `uses: actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `action.yml:333`
- `action.yml:337`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings in hardened/action/action.yml:
1. unpinned-uses: Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5 and actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4.
2. script-injection: Replaced both ${{ github.action_path }} expressions in run: blocks with the $GITHUB_ACTION_PATH environment variable (in the gh-graph install step and the recommend step).
3. github-env-injection: Added sanitization (printf '%s' ... | tr -d '\n\r') for ZAI_API_KEY and MOONSHOT_API_KEY before writing ANTHROPIC_AUTH_TOKEN to $GITHUB_ENV.
4. static-unsanitized-env-write (also covers the third github-env-injection location): Added sanitization for INPUT_MODEL before writing ANTHROPIC_MODEL to $GITHUB_ENV.

