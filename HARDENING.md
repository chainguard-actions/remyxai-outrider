<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.39

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.39** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable tag refs instead of full 40-character commit SHA digests, making the action vulnerable to supply-chain attacks if the upstream action tag is moved or compromised. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`. These should be pinned to their full SHA, e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `action.yml:581`
- `action.yml:585`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` — a GitHub Actions expression — inside shell command strings. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it, bypassing shell quoting. Offending lines: (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the 'Install gh-graph selection-pass tool on PATH' step; (2) `python ${{ github.action_path }}/src/run.py` in the 'Recommend + implement + open PR' step. The safe pattern is to use the `$GITHUB_ACTION_PATH` environment variable instead: `install -m 0755 "$GITHUB_ACTION_PATH/src/gh_graph.py"` and `python "$GITHUB_ACTION_PATH/src/run.py"`.

Locations:

- `action.yml:607`
- `action.yml:830`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes inherited process env vars directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Three unsanitized writes are present: (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `$ZAI_API_KEY` is set by the calling workflow's env block and is workflow-controlled; (2) `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — same for `$MOONSHOT_API_KEY`; (3) `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `$INPUT_MODEL` is derived from `inputs.model` via the step's env block. A newline embedded in any of these values would allow an attacker to inject arbitrary environment variable assignments into subsequent steps. The fix is to sanitize before writing: `safe=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r'); echo "ANTHROPIC_AUTH_TOKEN=$safe" >> "$GITHUB_ENV"`.

Locations:

- `action.yml:730`
- `action.yml:740`
- `action.yml:760`

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
2. script-injection: Replaced both ${{ github.action_path }} expressions in run: blocks with $GITHUB_ACTION_PATH (the equivalent environment variable), in the 'Install gh-graph' step and the 'Recommend + implement + open PR' step.
3. github-env-injection + static-unsanitized-env-write: Added printf/tr sanitization before all three $GITHUB_ENV writes: ZAI_API_KEY → safe_zai, MOONSHOT_API_KEY → safe_moonshot, and INPUT_MODEL → safe_model, each using `printf '%s' "$VAR" | tr -d '\n\r'` before the echo write.

