<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.41

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.41** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` (a GitHub Actions expression) inside shell command strings, violating rule (a). Any `${{ ... }}` expression inside a `run:` block is a script-injection risk regardless of context. (1) The 'Install gh-graph selection-pass tool on PATH' step uses: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) The 'Recommend + implement + open PR' step uses: `python ${{ github.action_path }}/src/run.py`. Both should use the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:265`
- `action.yml:430`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes workflow-controlled (caller-supplied) environment variables to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Three unsanitized writes: (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `$ZAI_API_KEY` is an inherited process env var set by the calling workflow; (2) `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — same issue with `$MOONSHOT_API_KEY`; (3) `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `$INPUT_MODEL` is sourced from `${{ inputs.model }}` via the step's env block. A newline in any of these values could inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:351`
- `action.yml:362`
- `action.yml:395`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml composite steps use mutable version tags instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved: (1) `uses: actions/setup-python@v5` — should be pinned to a full SHA; (2) `uses: actions/setup-node@v4` — should be pinned to a full SHA.

Locations:

- `action.yml:237`
- `action.yml:242`

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
2. script-injection: Replaced both occurrences of ${{ github.action_path }} in run: blocks with $GITHUB_ACTION_PATH (in 'Install gh-graph selection-pass tool on PATH' and 'Recommend + implement + open PR' steps).
3. github-env-injection + static-unsanitized-env-write: Added printf '%s' ... | tr -d '\n\r' sanitization before all three $GITHUB_ENV writes: ZAI_API_KEY → safe_zai_key, MOONSHOT_API_KEY → safe_moonshot_key, and INPUT_MODEL → safe_model.

