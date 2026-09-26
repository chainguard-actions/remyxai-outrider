<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.40

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.40** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two run: blocks in action.yml directly interpolate ${{ github.action_path }} inside shell command strings (rule a). Any ${{ ... }} expression inside a run: block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it. Offending lines: (1) 'Install gh-graph selection-pass tool on PATH' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`; (2) 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py`. Both should use the $GITHUB_ACTION_PATH environment variable instead.

Locations:

- `action.yml:399`
- `action.yml:556`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes inherited process env vars to $GITHUB_ENV without sanitization. Composite actions inherit env from the calling workflow, so $ZAI_API_KEY, $MOONSHOT_API_KEY, and $INPUT_MODEL are workflow-controlled and must be treated as untrusted. The writes `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"`, `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"`, and `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` all lack the required sanitization step (`safe=$(printf '%s' "$VAR" | tr -d '\n\r')`) before writing to the special environment file, allowing newline injection that could set arbitrary environment variables.

Locations:

- `action.yml:490`
- `action.yml:502`
- `action.yml:519`

### unpinned-uses (severity: high)

Two composite action steps use mutable version tags instead of pinned full 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream action tags are moved or compromised. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`. These should be pinned to their full commit SHAs (e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`).

Locations:

- `action.yml:380`
- `action.yml:385`

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
2. script-injection: Replaced both ${{ github.action_path }} interpolations with $GITHUB_ACTION_PATH environment variable references (lines 399 and 556).
3. github-env-injection: Added printf/tr sanitization for $ZAI_API_KEY, $MOONSHOT_API_KEY, and $INPUT_MODEL before writing to $GITHUB_ENV to prevent newline injection attacks.
4. static-unsanitized-env-write: Same fix as github-env-injection — $INPUT_MODEL is now sanitized via safe_model=$(printf '%s' "$INPUT_MODEL" | tr -d '\n\r') before the echo to $GITHUB_ENV.

