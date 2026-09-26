<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.43

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.43** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two composite action steps use mutable version tags instead of pinned full SHA digests, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`.

Locations:

- `action.yml:490`
- `action.yml:494`

### script-injection (severity: high)

Rule (a): `${{ github.action_path }}` is interpolated directly inside `run:` shell command strings in two steps. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it. Offending lines: (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the 'Install gh-graph selection-pass tool on PATH' step; (2) `python ${{ github.action_path }}/src/run.py` in the 'Recommend + implement + open PR' step.

Locations:

- `action.yml:519`
- `action.yml:714`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes workflow-controlled inherited process env vars to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Three unsanitized writes: (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `ZAI_API_KEY` is set by the calling workflow's `env:` block and is untrusted; (2) `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — same issue with `MOONSHOT_API_KEY`; (3) `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `INPUT_MODEL` is sourced from `${{ inputs.model }}` via the step's `env:` block. A newline in any of these values could inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:637`
- `action.yml:648`
- `action.yml:670`

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
3. Moved ${{ github.action_path }} from run: blocks to env: blocks (as ACTION_PATH) in both the 'Install gh-graph' step and the 'Recommend + implement + open PR' step; the latter's ACTION_PATH was merged into the existing env: block.
4. Sanitized ZAI_API_KEY and MOONSHOT_API_KEY before writing to $GITHUB_ENV using printf '%s' ... | tr -d '\n\r'.
5. Sanitized INPUT_MODEL before writing ANTHROPIC_MODEL to $GITHUB_ENV using printf '%s' ... | tr -d '\n\r'.

