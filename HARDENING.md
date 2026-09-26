<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.44

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.44** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` — a GitHub Actions expression — inside shell command strings. Per the script-injection check, ANY `${{ ... }}` expression inside a `run:` block is a finding, including `runner.*`/`github.*` context values that flow through YAML template substitution before the shell sees them.

(a) In the 'Install gh-graph selection-pass tool on PATH' step:
  `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`

(b) In the 'Recommend + implement + open PR' step:
  `python ${{ github.action_path }}/src/run.py`

Fix: use the `$GITHUB_ACTION_PATH` environment variable instead, which is already available as a shell variable and does not require expression interpolation.

Locations:

- `action.yml:430`
- `action.yml:556`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes inherited process environment variables directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). These variables are set by the calling workflow's `env:` block and are therefore workflow-controlled (untrusted) inputs. Three unsanitized writes:

1. `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` (zai branch)
2. `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` (moonshot branch)
3. `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` (model name forwarding)

A newline embedded in any of these values would allow an attacker to inject arbitrary environment variable assignments into subsequent steps. Fix: sanitize each value with `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before writing to `$GITHUB_ENV`.

Locations:

- `action.yml:490`
- `action.yml:505`
- `action.yml:530`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling supply-chain attacks.

- `uses: actions/setup-python@v5` (should be pinned to a full SHA, e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`)
- `uses: actions/setup-node@v4` (should be pinned to a full SHA, e.g. `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`)

Locations:

- `action.yml:395`
- `action.yml:400`

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
2. script-injection: Replaced both ${{ github.action_path }} expressions in run: blocks with $GITHUB_ACTION_PATH (the equivalent shell environment variable).
3. github-env-injection: Sanitized ZAI_API_KEY and MOONSHOT_API_KEY with 'safe_X=$(printf "%s" "$X" | tr -d "\n\r")' before writing ANTHROPIC_AUTH_TOKEN to $GITHUB_ENV.
4. static-unsanitized-env-write: Sanitized INPUT_MODEL with the same printf/tr pattern before writing ANTHROPIC_MODEL to $GITHUB_ENV.

