<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.9.0** was hardened automatically. 12 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Composite action steps use actions by mutable tag instead of a full 40-char commit SHA: `uses: actions/setup-python@v7` and `uses: actions/setup-node@v7`.

Locations:

- `action.yml:576`
- `action.yml:600`

### unsafe-shell (severity: high)

Remote installer is piped straight into a shell in the 'Install the coding-agent CLI' step (backboard branch): `curl -fsSL https://app.backboard.io/api/cli | sh`. Download it to a file, verify it, then run it as a separate step.

Locations:

- `action.yml:651`

### script-injection (severity: high)

Rule (a): step 'Configure the model backend' puts an expression directly in run: `run: python ${{ github.action_path }}/src/configure_backend.py`.

Locations:

- `action.yml:585`

### script-injection (severity: high)

Rule (a): step 'Resolve the Node line from package.json' puts an expression directly in run: `... "${{ github.action_path }}/package.json")"`.

Locations:

- `action.yml:595`

### script-injection (severity: high)

Rule (a): step 'Install the coding-agent CLI' puts an expression directly in run: `PKG="${{ github.action_path }}/package.json"`.

Locations:

- `action.yml:615`

### script-injection (severity: high)

Rule (a): step 'Install pytest + remyxai CLI + pyyaml' puts an expression directly in run: `run: pip install -r "${{ github.action_path }}/requirements-action.txt"`.

Locations:

- `action.yml:680`

### script-injection (severity: high)

Rule (a): step 'Install pypdf' puts an expression directly in run: `run: pip install "$(grep -E '^pypdf' "${{ github.action_path }}/requirements-action-optional.txt")"`.

Locations:

- `action.yml:689`

### script-injection (severity: high)

Rule (a): step 'Install gh-graph selection-pass tool on PATH' puts an expression directly in run: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`.

Locations:

- `action.yml:698`

### script-injection (severity: high)

Rule (a): step 'Install cocoindex-code AST search' puts expressions directly in run: `pipx install "$(grep -E '^cocoindex-code' "${{ github.action_path }}/requirements-action-optional.txt")"` and `SKILLS_HOME=$(python "${{ github.action_path }}/src/agent_tooling.py" --skills-home)`.

Locations:

- `action.yml:719`

### script-injection (severity: high)

Rule (a): step 'Write ENVIRONMENTS.md' puts an expression directly in run: `python "${{ github.action_path }}/src/agent_tooling.py" --environments-md > ENVIRONMENTS.md`.

Locations:

- `action.yml:762`

### script-injection (severity: high)

Rule (a): step 'Recommend + implement + open PR' puts an expression directly in run: `python ${{ github.action_path }}/src/run.py`. Pass it through env (e.g. use $GITHUB_ACTION_PATH) and double-quote it.

Locations:

- `action.yml:845`

### github-env-injection (severity: high)

Step 'Install the coding-agent CLI' writes `dirname "$BB" >> "$GITHUB_PATH"` with no `printf '%s' ... | tr -d '\n\r'` sanitization. BB comes from inherited process env ($RUNNER_TEMP via BACKBOARD_INSTALL, or `command -v backboard`, which depends on the caller's PATH), so the composite action treats it as untrusted.

Locations:

- `action.yml:661`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, script-injection, github-env-injection

**Notes:**

Pinned actions/setup-python@6d4fd22b20465ccc0f7354476588e8df7c9ca0e0 (v7) and actions/setup-node@949feb2413d6458794dcd2491c4babbbce0c15c1 (v7). Replaced every `${{ github.action_path }}` in a run: script with a double-quoted "$GITHUB_ACTION_PATH" (configure_backend.py, package.json in two steps, both requirements files, gh_graph.py, agent_tooling.py twice, run.py). The backboard installer is now downloaded with curl to a mktemp file under $RUNNER_TEMP, checked to be non-empty, run with `sh "$file"` (no arguments, same as upstream; BACKBOARD_INSTALL is still exported), then deleted. The directory written to GITHUB_PATH now has CR/LF removed with `printf '%s' | tr -d '\n\r'` first. Several edits garbled nearby lines (whitespace and line breaks); I fixed and re-checked each one. Nothing was run, so the changes have not been tested.

