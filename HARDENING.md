<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.53

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.53** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml's composite action steps use mutable tag refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream action tags are moved:
- `uses: actions/setup-python@v5` (tag ref)
- `uses: actions/setup-node@v4` (tag ref)

Locations:

- `action.yml:476`
- `action.yml:480`

### script-injection (severity: high)

Two `run:` blocks directly interpolate `${{ github.action_path }}` inside shell command strings (sub-rule a). Any `${{ ... }}` expression interpolated directly in a `run:` block is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it, bypassing shell quoting.

1. In the 'Install gh-graph selection-pass tool on PATH' step:
   `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`

2. In the 'Recommend + implement + open PR' step:
   `python ${{ github.action_path }}/src/run.py`

Both should use the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:497`
- `action.yml:591`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes workflow-controlled (inherited) process environment variables directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Composite actions inherit env from the calling workflow, so `$ZAI_API_KEY`, `$MOONSHOT_API_KEY`, and `$INPUT_MODEL` are workflow-controlled and must be treated as untrusted. A newline embedded in any of these values could inject arbitrary environment variables into subsequent steps.

Failing lines:
- `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` (zai branch)
- `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` (moonshot branch)
- `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` (model name write)

Each should be preceded by: `safe=$(printf '%s' "$VAR" | tr -d '\n\r')`

Locations:

- `action.yml:543`
- `action.yml:551`
- `action.yml:563`

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
2. script-injection: Replaced both ${{ github.action_path }} interpolations in run: blocks with $GITHUB_ACTION_PATH (in the gh-graph install step and the run.py invocation).
3. github-env-injection: Added safe_zai=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r') and safe_moonshot=$(printf '%s' "$MOONSHOT_API_KEY" | tr -d '\n\r') before writing ANTHROPIC_AUTH_TOKEN to $GITHUB_ENV in the zai and moonshot branches.
4. static-unsanitized-env-write: Added safe_model=$(printf '%s' "$INPUT_MODEL" | tr -d '\n\r') before writing ANTHROPIC_MODEL to $GITHUB_ENV.

