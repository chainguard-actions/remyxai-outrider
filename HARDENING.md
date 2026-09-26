<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.38

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.38** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml pin to mutable version tags instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved:
- `uses: actions/setup-python@v5`
- `uses: actions/setup-node@v4`
These should be pinned to their full commit SHAs (e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`).

Locations:

- `action.yml:430`
- `action.yml:434`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks directly interpolate a `${{ ... }}` GitHub Actions expression inside a shell command string. Any expression inside a `run:` block is subject to template substitution before the shell sees it, creating a script-injection risk.

1. In the "Install gh-graph selection-pass tool on PATH" step:
   `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`

2. In the "Recommend + implement + open PR" step:
   `python ${{ github.action_path }}/src/run.py`

Both should use the `$GITHUB_ACTION_PATH` environment variable instead (e.g. `python "$GITHUB_ACTION_PATH/src/run.py"`).

Locations:

- `action.yml:461`
- `action.yml:619`

### github-env-injection (severity: high)

The "Configure backend from provider input" step writes inherited process environment variables — set by the calling workflow's `env:` block and therefore workflow-controlled — directly to `$GITHUB_ENV` without the required newline-stripping sanitization (`printf '%s' "$VAR" | tr -d '\n\r'`). A caller could inject newlines into these values to write arbitrary key=value pairs into the runner environment for subsequent steps.

Affected writes:
- `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` (zai branch)
- `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` (moonshot branch)
- `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` (model override)

`INPUT_MODEL` is sourced from `${{ inputs.model }}` via the step's `env:` block, and `ZAI_API_KEY`/`MOONSHOT_API_KEY` are inherited from the calling workflow's environment — all are untrusted for the purposes of this check. Each write should be preceded by sanitization, e.g.:
```
safe=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r')
echo "ANTHROPIC_AUTH_TOKEN=$safe" >> "$GITHUB_ENV"
```

Locations:

- `action.yml:531`
- `action.yml:541`
- `action.yml:560`

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
2. script-injection: Replaced both ${{ github.action_path }} occurrences in run: blocks with $GITHUB_ACTION_PATH (the safe environment variable equivalent).
3. github-env-injection: Added printf/tr sanitization before all three $GITHUB_ENV writes: ZAI_API_KEY (safe_zai), MOONSHOT_API_KEY (safe_moonshot), and INPUT_MODEL (safe_model).
4. static-unsanitized-env-write: Covered by the same INPUT_MODEL sanitization fix applied for github-env-injection.

