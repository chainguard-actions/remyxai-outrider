<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.45

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.45** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved:
- `uses: actions/setup-python@v5` (tag: v5)
- `uses: actions/setup-node@v4` (tag: v4)
These should be pinned to their full commit SHAs, e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `action.yml:396`
- `action.yml:400`

### script-injection (severity: high)

Two `run:` blocks directly interpolate `${{ github.action_path }}` inside shell command strings (rule a). Any `${{ ... }}` expression interpolated directly in a `run:` block is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it, bypassing shell quoting. Offending lines:
1. `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` — in the 'Install gh-graph selection-pass tool on PATH' step.
2. `python ${{ github.action_path }}/src/run.py` — in the 'Recommend + implement + open PR' step.
Fix: use the `$GITHUB_ACTION_PATH` environment variable instead (already available in composite actions), e.g. `install -m 0755 "$GITHUB_ACTION_PATH/src/gh_graph.py"` and `python "$GITHUB_ACTION_PATH/src/run.py"`.

Locations:

- `action.yml:420`
- `action.yml:560`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes inherited process environment variables directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). These env vars (`ZAI_API_KEY`, `MOONSHOT_API_KEY`, `INPUT_MODEL`) are set by the calling workflow's `env:` block and are therefore workflow-controlled (untrusted) inputs. Writing them unsanitized to GITHUB_ENV allows a newline-injection attack that could define arbitrary additional environment variables for subsequent steps. Affected writes:
- `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` (zai branch)
- `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` (moonshot branch)
- `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` (model name branch)
Fix: sanitize each value before writing, e.g.:
```
safe=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r')
echo "ANTHROPIC_AUTH_TOKEN=$safe" >> "$GITHUB_ENV"
```

Locations:

- `action.yml:480`
- `action.yml:490`
- `action.yml:510`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings in hardened/action/action.yml:
1. Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5
2. Pinned actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4
3. Replaced ${{ github.action_path }} with $GITHUB_ACTION_PATH in both run: blocks (Install gh-graph step and Recommend + implement + open PR step)
4. Sanitized ZAI_API_KEY before writing ANTHROPIC_AUTH_TOKEN to $GITHUB_ENV (safe_zai=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r'))
5. Sanitized MOONSHOT_API_KEY before writing ANTHROPIC_AUTH_TOKEN to $GITHUB_ENV (safe_moonshot=$(printf '%s' "$MOONSHOT_API_KEY" | tr -d '\n\r'))
6. Sanitized INPUT_MODEL before writing ANTHROPIC_MODEL to $GITHUB_ENV (safe_model=$(printf '%s' "$INPUT_MODEL" | tr -d '\n\r'))

