<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.37

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.37** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings. Any `${{ ... }}` expression interpolated directly inside a `run:` block is a script-injection risk — the expression is substituted by the Actions template engine before the shell ever sees the string, bypassing shell quoting. (1) In the 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) In the 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py`. Both should use the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:310`
- `action.yml:460`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes inherited process environment variables directly to `$GITHUB_ENV` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). These env vars are set by the calling workflow and are workflow-controlled (untrusted). Three unsanitized writes: (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — ZAI_API_KEY is an inherited env var from the caller. (2) `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — MOONSHOT_API_KEY is an inherited env var from the caller. (3) `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — INPUT_MODEL is set from `inputs.model` via the step's env: block. A newline embedded in any of these values could inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:390`
- `action.yml:400`
- `action.yml:415`

### unpinned-uses (severity: high)

Two `uses:` references in the composite action's `runs.steps` are pinned to mutable version tags rather than immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or the repository is compromised. Failing references: (1) `uses: actions/setup-python@v5` — should be pinned to a full SHA, e.g. `actions/setup-python@<40-char-sha> # v5`. (2) `uses: actions/setup-node@v4` — should be pinned to a full SHA, e.g. `actions/setup-node@<40-char-sha> # v4`.

Locations:

- `action.yml:274`
- `action.yml:279`

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
2. script-injection: Replaced both `${{ github.action_path }}` occurrences in run: blocks with `$GITHUB_ACTION_PATH` (the safe environment variable equivalent).
3. github-env-injection: Added `printf '%s' ... | tr -d '\n\r'` sanitization before writing ZAI_API_KEY (as ANTHROPIC_AUTH_TOKEN), MOONSHOT_API_KEY (as ANTHROPIC_AUTH_TOKEN), and INPUT_MODEL (as ANTHROPIC_MODEL) to $GITHUB_ENV.
4. static-unsanitized-env-write: Same fix as the INPUT_MODEL sanitization above — the ANTHROPIC_MODEL write now uses safe_model=$(printf '%s' "$INPUT_MODEL" | tr -d '\n\r') before echoing to $GITHUB_ENV.

