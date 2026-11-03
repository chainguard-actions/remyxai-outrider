<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.9.1** was hardened automatically. 12 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two composite steps reference actions by a mutable tag instead of a 40-character commit SHA: `uses: actions/setup-python@v7` and `uses: actions/setup-node@v7`.

Locations:

- `action.yml:700`
- `action.yml:730`

### unsafe-shell (severity: high)

The 'Install the coding-agent CLI' step pipes remote content straight into a shell: `curl -fsSL https://app.backboard.io/api/cli | sh`. Download the installer to a file, check it (for example with a pinned checksum), then run it as a separate command.

Locations:

- `action.yml:775`

### script-injection (severity: high)

Sub-rule (a), step 'Configure the model backend': an expression is placed directly in the run script: `run: python ${{ github.action_path }}/src/configure_backend.py`. Use "$GITHUB_ACTION_PATH" instead.

Locations:

- `action.yml:718`

### script-injection (severity: high)

Sub-rule (a), step 'Resolve the Node line from package.json': `... "${{ github.action_path }}/package.json")"` puts an expression directly in the run script. Use "$GITHUB_ACTION_PATH".

Locations:

- `action.yml:726`

### script-injection (severity: high)

Sub-rule (a), step 'Install the coding-agent CLI': `PKG="${{ github.action_path }}/package.json"` puts an expression directly in the run script.

Locations:

- `action.yml:743`

### script-injection (severity: high)

Sub-rule (a), step 'Install pytest + remyxai CLI + pyyaml': `run: pip install -r "${{ github.action_path }}/requirements-action.txt"` puts an expression directly in the run script.

Locations:

- `action.yml:800`

### script-injection (severity: high)

Sub-rule (a), step 'Install pypdf': `run: pip install "$(grep -E '^pypdf' "${{ github.action_path }}/requirements-action-optional.txt")"` puts an expression directly in the run script.

Locations:

- `action.yml:808`

### script-injection (severity: high)

Sub-rule (a), step 'Install gh-graph selection-pass tool on PATH': `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph` puts an expression directly in the run script.

Locations:

- `action.yml:816`

### script-injection (severity: high)

Sub-rule (a), step 'Install cocoindex-code AST search': `pipx install "$(grep -E '^cocoindex-code' "${{ github.action_path }}/requirements-action-optional.txt")"` and `SKILLS_HOME=$(python "${{ github.action_path }}/src/agent_tooling.py" --skills-home)` both put an expression directly in the run script.

Locations:

- `action.yml:840`

### script-injection (severity: high)

Sub-rule (a), step 'Write ENVIRONMENTS.md': `python "${{ github.action_path }}/src/agent_tooling.py" --environments-md > ENVIRONMENTS.md` puts an expression directly in the run script.

Locations:

- `action.yml:878`

### script-injection (severity: high)

Sub-rule (a), step 'Recommend + implement + open PR': `python ${{ github.action_path }}/src/run.py` puts an expression directly in the run script, and it is unquoted.

Locations:

- `action.yml:955`

### github-env-injection (severity: high)

Step 'Install the coding-agent CLI' (backboard branch): `dirname "$BB" >> "$GITHUB_PATH"` writes a value with no `printf '%s' ... | tr -d '\n\r'` step first. BB comes from BACKBOARD_INSTALL="$RUNNER_TEMP/backboard", and RUNNER_TEMP is inherited from the process environment rather than set by this script, or BB falls back to `command -v backboard`. Sanitize the value before appending it to GITHUB_PATH.

Locations:

- `action.yml:785`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, script-injection, github-env-injection

**Notes:**

- Pinned actions/setup-python@v7 to 6d4fd22b20465ccc0f7354476588e8df7c9ca0e0 and actions/setup-node@v7 to 949feb2413d6458794dcd2491c4babbbce0c15c1. Each keeps a `# v7` comment.
- Every `${{ github.action_path }}` inside a run script is now "$GITHUB_ACTION_PATH" in double quotes. That covers configure_backend, the node-line step, PKG, the two requirements installs, gh_graph, the cocoindex install, agent_tooling and run.py. All of these steps use bash.
- The backboard installer now downloads to a mktemp file and stops with an error if that file is empty. It then runs with `sh "$file"` (no arguments, so there was no `--` to drop) and the file is deleted. I did not add a checksum check: upstream publishes no checksum, and the installer endpoint is not versioned.
- The directory written to GITHUB_PATH now has newlines and carriage returns stripped (`tr -d '\n\r'`) before it is appended.

### Iteration 2

**Fixes applied:** unpinned-uses, invalid-yaml

**Notes:**

Pinned actions/setup-python@v7 to its commit SHA, 6d4fd22b20465ccc0f7354476588e8df7c9ca0e0, which lookup_action_sha returned for that tag. The tag is kept as a '# v7' comment. Fixed two corrupted spots in action.yml that broke YAML parsing: (1) In the pypdf step, `continue-on-error: tru` had merged into the `run:` line and the command had an extra `")"` on the end. I split them back onto two lines as `continue-on-error: true` and `run: pip install "$(grep -E '^pypdf' ...)"`. (2) In the ENVIRONMENTS.md step, the script was garbled: there was a leftover piece of a `${{ github.action_path }}` path and the opening `if` line was missing. I rebuilt it as `if [ ! -e ENVIRONMENTS.md ]; then python "$GITHUB_ACTION_PATH/src/agent_tooling.py" --environments-md > ENVIRONMENTS.md ...`. The `else`/`fi` branch that followed (leave the repo's own file alone) was still intact. The exact original test (`-e` or `-f`) isn't certain, because no other file in the repo shows the original condition; the two behave the same for a normal file. No other changes.

