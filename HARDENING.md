<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.49

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.7.49** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`.

Locations:

- `action.yml:435`
- `action.yml:440`

### script-injection (severity: high)

Rule (a): Two `run:` blocks directly interpolate `${{ github.action_path }}` inside shell command strings. Any `${{ ... }}` expression interpolated directly in a `run:` block is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it. Offending lines: (1) `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` in the 'Install gh-graph selection-pass tool on PATH' step; (2) `python ${{ github.action_path }}/src/run.py` in the 'Recommend + implement + open PR' step. Fix: use the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:470`
- `action.yml:600`

### github-env-injection (severity: high)

The 'Configure backend from provider input' step writes inherited process env vars to `$GITHUB_ENV` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). These env vars are set by the calling workflow and are therefore workflow-controlled (untrusted) inputs. Three unsanitized writes: (1) `echo "ANTHROPIC_AUTH_TOKEN=$ZAI_API_KEY" >> "$GITHUB_ENV"` — `ZAI_API_KEY` is an inherited env var from the calling workflow; (2) `echo "ANTHROPIC_AUTH_TOKEN=$MOONSHOT_API_KEY" >> "$GITHUB_ENV"` — `MOONSHOT_API_KEY` is an inherited env var from the calling workflow; (3) `echo "ANTHROPIC_MODEL=$INPUT_MODEL" >> "$GITHUB_ENV"` — `INPUT_MODEL` is sourced from `inputs.model` (workflow-controlled). A newline in any of these values could inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:533`
- `action.yml:543`
- `action.yml:555`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_MODEL in step "Configure backend from provider input" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:665`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all four findings in hardened/action/action.yml: (1) Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065 and actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, preserving original tags as comments. (2) Replaced both ${{ github.action_path }} interpolations in run: blocks with $GITHUB_ACTION_PATH environment variable. (3) Sanitized ZAI_API_KEY, MOONSHOT_API_KEY, and INPUT_MODEL before writing to $GITHUB_ENV using printf '%s' ... | tr -d '\n\r' pattern to prevent newline injection attacks.

