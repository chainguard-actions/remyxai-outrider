<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.51

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.51** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks directly interpolate `${{ github.action_path }}` — a `${{ ... }}` expression — inside shell command strings. Per the script-injection check, ANY `${{ ... }}` expression directly inside a `run:` block is a violation (sub-rule a), regardless of which context it reads from. (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the 'Install gh-graph' step. (2) `python ${{ github.action_path }}/src/run.py` in the 'Recommend + implement + open PR' step. These should be replaced with the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:25932`
- `action.yml:36371`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes workflow-controlled (inherited) process env vars to `$GITHUB_ENV` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). Specifically: (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `$ZAI_API_KEY` is set by the calling workflow's `env:` block and is untrusted. (2) `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — same issue with `$MOONSHOT_API_KEY`. (3) `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `$INPUT_MODEL` is derived from `inputs.model` via the `env:` block. A newline injected into any of these values could write arbitrary variables into `$GITHUB_ENV`, affecting subsequent steps.

Locations:

- `action.yml:31851`
- `action.yml:32289`
- `action.yml:33351`

### unpinned-uses (severity: high)

Two `uses:` references in the composite action steps use mutable version tags instead of full 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`. These should be pinned to their full commit SHAs (e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`).

Locations:

- `action.yml:24582`
- `action.yml:24708`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5 and actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4. (2) Replaced both `${{ github.action_path }}` expressions with `$GITHUB_ACTION_PATH` in the 'Install gh-graph' and 'Recommend + implement + open PR' run: blocks. (3) Sanitized ZAI_API_KEY, MOONSHOT_API_KEY, and INPUT_MODEL before writing to $GITHUB_ENV using `safe_var=$(printf '%s' "$VAR" | tr -d '\n\r')` pattern to prevent newline injection attacks.

