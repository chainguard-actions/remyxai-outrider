<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.48

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.48** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings. Per the script-injection check, ANY `${{ ... }}` expression interpolated directly inside a `run:` block is a violation (sub-rule a), regardless of which context it reads from. (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the 'Install gh-graph selection-pass tool on PATH' step. (2) `python ${{ github.action_path }}/src/run.py` in the 'Recommend + implement + open PR' step.

Locations:

- `action.yml:476`
- `action.yml:567`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step in action.yml writes inherited process env vars and input-derived env vars to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Three unsanitized writes: (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `$ZAI_API_KEY` is a workflow-controlled inherited env var; (2) `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — `$MOONSHOT_API_KEY` is a workflow-controlled inherited env var; (3) `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `$INPUT_MODEL` is set from `inputs.model` (caller-controlled). A newline injected into any of these values could add arbitrary entries to the runner's environment for subsequent steps.

Locations:

- `action.yml:519`
- `action.yml:527`
- `action.yml:535`
- `action.yml:543`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved. Failing references: `uses: actions/setup-python@v5` and `uses: actions/setup-node@v4`. These should be pinned to their full SHA digests (e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`).

Locations:

- `action.yml:436`
- `action.yml:440`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings in hardened/action/action.yml:

1. unpinned-uses: Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 and actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 with tag comments.

2. script-injection (2 locations): Moved ${{ github.action_path }} out of run: blocks into env: blocks as ACTION_PATH, then referenced via $ACTION_PATH in the shell scripts.

3. github-env-injection (3 locations): Added printf '%s' ... | tr -d '\n\r' sanitization before writing ZAI_API_KEY, MOONSHOT_API_KEY, and INPUT_MODEL to $GITHUB_ENV.

4. static-unsanitized-env-write: Covered by the INPUT_MODEL sanitization fix above (same location as finding #3's third instance).

