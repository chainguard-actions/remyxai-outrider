<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.34

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.34** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings. Any `${{ ... }}` expression inside a `run:` script is a script-injection risk because the value is substituted by the GitHub Actions template engine before the shell ever sees it. (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the 'Install gh-graph selection-pass tool on PATH' step. (2) `python ${{ github.action_path }}/src/run.py` in the 'Recommend + implement + open PR' step. Both should use the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:375`
- `action.yml:490`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes workflow-controlled (inherited) process environment variables to `$GITHUB_ENV` without sanitization (`printf '%s' ... | tr -d '\n\r'`). As a composite action, it inherits env vars from the calling workflow, making `$ZAI_API_KEY`, `$MOONSHOT_API_KEY`, and `$INPUT_MODEL` (sourced from `inputs.model`) untrusted for injection purposes. Specifically: (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — unsanitized `$ZAI_API_KEY`. (2) `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — unsanitized `$MOONSHOT_API_KEY`. (3) `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `$INPUT_MODEL` is set from `inputs.model` via the step's `env:` block. A newline in any of these values could inject arbitrary variables into `$GITHUB_ENV`.

Locations:

- `action.yml:430`
- `action.yml:440`
- `action.yml:455`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised. Failing references: (1) `uses: actions/setup-python@v5` — should be pinned to a full SHA. (2) `uses: actions/setup-node@v4` — should be pinned to a full SHA.

Locations:

- `action.yml:352`
- `action.yml:357`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:636`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings in action.yml: (1) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065 and actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020. (2) Replaced both ${{ github.action_path }} expressions in run: blocks with $GITHUB_ACTION_PATH environment variable. (3) Sanitized $ZAI_API_KEY, $MOONSHOT_API_KEY, and $INPUT_MODEL before writing to $GITHUB_ENV using printf '%s' ... | tr -d '\n\r' to prevent newline injection attacks.

