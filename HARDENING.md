<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.9.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.9.2** was hardened automatically. 13 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two composite steps pin their actions to a version tag instead of a full commit SHA: `uses: actions/setup-python@v7` (line 634) and `uses: actions/setup-node@v7` (line 670). Pin each one to its 40-character commit SHA.

Locations:

- `action.yml:634`
- `action.yml:670`

### unsafe-shell (severity: high)

The 'Install the coding-agent CLI' step, on the backboard branch, downloads a remote installer and pipes it straight into a shell: `curl -fsSL https://app.backboard.io/api/cli | sh`. Download it to a file first, check it if possible, then run it as a separate command.

Locations:

- `action.yml:721`

### script-injection (severity: high)

Rule (a): the 'Configure the model backend' step puts an expression directly in run: `run: python ${{ github.action_path }}/src/configure_backend.py`. The expression is also unquoted. Use "$GITHUB_ACTION_PATH" in double quotes instead.

Locations:

- `action.yml:655`

### script-injection (severity: high)

Rule (a): the 'Resolve the Node line from package.json' step puts an expression directly in run: `spec="$(python -c '...' "${{ github.action_path }}/package.json")"`. Use "$GITHUB_ACTION_PATH" instead.

Locations:

- `action.yml:665`

### script-injection (severity: high)

Rule (a): the 'Install the coding-agent CLI' step puts an expression directly in run: `PKG="${{ github.action_path }}/package.json"`. Use "$GITHUB_ACTION_PATH" instead.

Locations:

- `action.yml:686`

### script-injection (severity: high)

Rule (a): the 'Install pytest + remyxai CLI + pyyaml' step puts an expression directly in run: `run: pip install -r "${{ github.action_path }}/requirements-action.txt"`.

Locations:

- `action.yml:750`

### script-injection (severity: high)

Rule (a): the 'Install pypdf' step puts an expression directly in run: `run: pip install "$(grep -E '^pypdf' "${{ github.action_path }}/requirements-action-optional.txt")"`.

Locations:

- `action.yml:759`

### script-injection (severity: high)

Rule (a): the 'Install gh-graph selection-pass tool on PATH' step puts an expression directly in run: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`.

Locations:

- `action.yml:768`

### script-injection (severity: high)

Rule (a): the 'Install cocoindex-code AST search' step puts expressions directly in run: `pipx install "$(grep -E '^cocoindex-code' "${{ github.action_path }}/requirements-action-optional.txt")"` and `SKILLS_HOME=$(python "${{ github.action_path }}/src/agent_tooling.py" --skills-home)`.

Locations:

- `action.yml:792`
- `action.yml:794`

### script-injection (severity: high)

Rule (a): the 'Write ENVIRONMENTS.md' step puts an expression directly in run: `python "${{ github.action_path }}/src/agent_tooling.py" \`.

Locations:

- `action.yml:835`

### script-injection (severity: high)

Rule (a): the 'Recommend + implement + open PR' step puts an expression directly in run, unquoted: `python ${{ github.action_path }}/src/run.py`. Use "$GITHUB_ACTION_PATH" in double quotes instead.

Locations:

- `action.yml:917`

### github-env-injection (severity: high)

The 'Resolve the Node line from package.json' step writes `spec` to $GITHUB_OUTPUT with no sanitization: `echo "spec=$spec" >> "$GITHUB_OUTPUT"`. `spec` is read from a file whose path comes from `${{ github.action_path }}`. Strip newlines before the write with `printf '%s' "$spec" | tr -d '\n\r'`.

Locations:

- `action.yml:666`

### github-env-injection (severity: high)

The 'Install the coding-agent CLI' step writes `dirname "$BB" >> "$GITHUB_PATH"` with no sanitization. BB is built from the inherited env var $RUNNER_TEMP (through BACKBOARD_INSTALL) or from `command -v backboard`, so it is not a literal set inside this script. Sanitize it with `printf '%s' ... | tr -d '\n\r'` before writing to GITHUB_PATH.

Locations:

- `action.yml:731`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, script-injection, github-env-injection

**Notes:**

Pinned actions/setup-python and actions/setup-node to v7 commit SHAs. Backboard installer is now downloaded to a mktemp file under $RUNNER_TEMP, checked for non-empty, then run with `sh` (BACKBOARD_INSTALL still exported). Replaced every ${{ github.action_path }} in run: scripts with quoted "$GITHUB_ACTION_PATH". Node spec is newline-stripped before writing to GITHUB_OUTPUT; backboard bin dir is newline-stripped before writing to GITHUB_PATH.

