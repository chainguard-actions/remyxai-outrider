<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.54

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.54** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` inside shell command strings. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk regardless of context. (1) In the 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`. (2) In the 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py`. These should be replaced with the equivalent pre-set env var `$GITHUB_ACTION_PATH`.

Locations:

- `action.yml:455`
- `action.yml:638`

### unpinned-uses (severity: high)

Two composite action steps use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved: `actions/setup-python@v5` and `actions/setup-node@v4`. These should be pinned to their full commit SHAs (e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`).

Locations:

- `action.yml:430`
- `action.yml:434`

### github-env-injection (severity: high)

In the 'Configure backend from provider input' step, inherited process environment variables sourced from the calling workflow are written to `$GITHUB_ENV` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). Specifically: (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `ZAI_API_KEY` is a workflow-controlled env var; (2) `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — `MOONSHOT_API_KEY` is a workflow-controlled env var; (3) `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `INPUT_MODEL` is set from `${{ inputs.model }}`, an attacker-controllable input. A newline in any of these values could inject arbitrary variables into subsequent steps' environments.

Locations:

- `action.yml:558`
- `action.yml:570`
- `action.yml:590`

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
2. script-injection: Replaced both occurrences of ${{ github.action_path }} in run: blocks with $GITHUB_ACTION_PATH (the pre-set env var equivalent).
3. github-env-injection (lines 558, 570): Sanitized ZAI_API_KEY and MOONSHOT_API_KEY with `printf '%s' "$VAR" | tr -d '\n\r'` before writing ANTHROPIC_AUTH_TOKEN to $GITHUB_ENV.
4. github-env-injection + static-unsanitized-env-write (lines 590, 665): Sanitized INPUT_MODEL with `printf '%s' "$INPUT_MODEL" | tr -d '\n\r'` before writing ANTHROPIC_MODEL to $GITHUB_ENV.

