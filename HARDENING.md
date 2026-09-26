<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.33

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.33** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved:
- `uses: actions/setup-python@v5` (tag `v5`)
- `uses: actions/setup-node@v4` (tag `v4`)
These should be pinned to their full commit SHAs.

Locations:

- `action.yml:335`
- `action.yml:340`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks directly interpolate `${{ github.action_path }}` — a `github.*` context expression — inside shell command strings. Any `${{ ... }}` expression directly inside a `run:` script is a script-injection finding regardless of which context it reads from.

Offending lines:
1. `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` (in the "Install gh-graph selection-pass tool on PATH" step)
2. `python ${{ github.action_path }}/src/run.py` (in the "Recommend + implement + open PR" step)

Fix: assign `github.action_path` to an env var (e.g. `ACTION_PATH: ${{ github.action_path }}`) and reference `"$ACTION_PATH"` in the shell script.

Locations:

- `action.yml:365`
- `action.yml:510`

### github-env-injection (severity: high)

The "Configure backend from provider input" step writes inherited process env vars and input-derived env vars directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). In a composite action, `ZAI_API_KEY`, `MOONSHOT_API_KEY`, and `INPUT_MODEL` are set by the calling workflow and must be treated as untrusted.

Offending writes:
1. `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `$ZAI_API_KEY` is a caller-supplied env var, unsanitized
2. `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — `$MOONSHOT_API_KEY` is a caller-supplied env var, unsanitized
3. `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `$INPUT_MODEL` derives from `inputs.model` via env, unsanitized

A newline injected into any of these values could add arbitrary key=value pairs to the runner's environment for subsequent steps.

Fix: sanitize each value before writing, e.g.:
```bash
safe=$(printf '%s' "$ZAI_API_KEY" | tr -d '\n\r')
echo "ANTHROPIC_AUTH_TOKEN=$safe" >> "$GITHUB_ENV"
```

Locations:

- `action.yml:430`
- `action.yml:440`
- `action.yml:460`

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
2. script-injection: Added env: ACTION_PATH: ${{ github.action_path }} to both the 'Install gh-graph' step and the 'Recommend + implement + open PR' step; replaced inline ${{ github.action_path }} with $ACTION_PATH in the run: scripts.
3. github-env-injection: Sanitized ZAI_API_KEY and MOONSHOT_API_KEY with printf '%s' ... | tr -d '\n\r' before writing ANTHROPIC_AUTH_TOKEN to $GITHUB_ENV.
4. static-unsanitized-env-write: Sanitized INPUT_MODEL with printf '%s' ... | tr -d '\n\r' before writing ANTHROPIC_MODEL to $GITHUB_ENV.

