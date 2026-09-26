<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.50

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.50** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings (rule a). Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it. (1) In the 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) In the 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py`. Both should use the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:460`
- `action.yml:583`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes inherited process environment variables to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). These variables are workflow-controlled (set by the calling workflow's `env:` block on the composite action step): (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `ZAI_API_KEY` is an inherited env var from the caller. (2) `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — `MOONSHOT_API_KEY` is an inherited env var from the caller. (3) `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `INPUT_MODEL` is mapped from `${{ inputs.model }}` in the step's `env:` block and then forwarded unsanitized. A newline injected into any of these values could add arbitrary variables to the runner environment.

Locations:

- `action.yml:510`
- `action.yml:521`
- `action.yml:543`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or the repository is compromised: (1) `uses: actions/setup-python@v5` — should be pinned to a full SHA, e.g. `actions/setup-python@<40-hex-sha> # v5`. (2) `uses: actions/setup-node@v4` — should be pinned to a full SHA, e.g. `actions/setup-node@<40-hex-sha> # v4`.

Locations:

- `action.yml:435`
- `action.yml:439`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings in hardened/action/action.yml:
1. unpinned-uses: Pinned actions/setup-python@v5 to @a26af69be951a213d495a4c3e4e4022e16d87065 # v5 and actions/setup-node@v4 to @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4.
2. script-injection: Replaced both ${{ github.action_path }} occurrences in run: blocks with $GITHUB_ACTION_PATH (in the 'Install gh-graph' step and the 'Recommend + implement + open PR' step).
3. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing ZAI_API_KEY (as ANTHROPIC_AUTH_TOKEN), MOONSHOT_API_KEY (as ANTHROPIC_AUTH_TOKEN), and INPUT_MODEL (as ANTHROPIC_MODEL) to $GITHUB_ENV.
4. static-unsanitized-env-write: Same fix as #3 for INPUT_MODEL — now sanitized with safe_model=$(printf '%s' "$INPUT_MODEL" | tr -d '\n\r') before the echo to $GITHUB_ENV.

