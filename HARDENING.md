<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.9.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml composite steps point to actions by a version tag instead of a full 40-character commit SHA: `uses: actions/setup-python@v7` (step 'Set up Python') and `uses: actions/setup-node@v7` (step 'Set up Node (for the agent CLI)'). Pin each one to its commit SHA, with the tag in a comment.

Locations:

- `action.yml:575`
- `action.yml:610`

### script-injection (severity: high)

Rule (a): the expression `${{ github.action_path }}` is placed directly inside the script of several run: blocks. Affected steps: 'Configure the model backend' (`run: python ${{ github.action_path }}/src/configure_backend.py`); 'Resolve the Node line from package.json' (`"${{ github.action_path }}/package.json"`); 'Install the coding-agent CLI' (`PKG="${{ github.action_path }}/package.json"`); 'Install pytest + remyxai CLI + pyyaml' (`pip install -r "${{ github.action_path }}/requirements-action.txt"`); 'Install pypdf' (`grep -E '^pypdf' "${{ github.action_path }}/requirements-action-optional.txt"`); 'Install gh-graph selection-pass tool on PATH' (`install -m 0755 "${{ github.action_path }}/src/gh_graph.py" ...`); 'Install cocoindex-code AST search' (pipx install / agent_tooling.py using `${{ github.action_path }}`); 'Write ENVIRONMENTS.md' (`python "${{ github.action_path }}/src/agent_tooling.py"`); 'Recommend + implement + open PR' (`python ${{ github.action_path }}/src/run.py`). Use the quoted environment variable "$GITHUB_ACTION_PATH" instead.

Locations:

- `action.yml:592`
- `action.yml:602`
- `action.yml:622`
- `action.yml:686`
- `action.yml:694`
- `action.yml:702`
- `action.yml:727`
- `action.yml:767`
- `action.yml:854`

### unsafe-shell (severity: high)

The step 'Install the coding-agent CLI' (backboard branch) pipes remote content straight into a shell: `curl -fsSL https://app.backboard.io/api/cli | sh`. Download the installer to a file first, ideally check its checksum, then run it as a separate command.

Locations:

- `action.yml:658`

### github-env-injection (severity: high)

Two run: blocks write values derived from inherited or github-context data to special environment files without the `printf '%s' ... | tr -d '\n\r'` sanitization step. (1) 'Install the coding-agent CLI': `dirname "$BB" >> "$GITHUB_PATH"`. $BB is derived from `BACKBOARD_INSTALL="$RUNNER_TEMP/backboard"`, where RUNNER_TEMP is inherited process env, or from `command -v backboard` output. (2) 'Resolve the Node line from package.json': `echo "spec=$spec" >> "$GITHUB_OUTPUT"`. $spec is read from a file located through `${{ github.action_path }}`. Both values are written unsanitized.

Locations:

- `action.yml:667`
- `action.yml:603`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, unsafe-shell, github-env-injection

**Notes:**

- unpinned-uses: actions/setup-python is now pinned to 6d4fd22b20465ccc0f7354476588e8df7c9ca0e0 (# v7) and actions/setup-node to 949feb2413d6458794dcd2491c4babbbce0c15c1 (# v7). Both SHAs came from lookup_action_sha.
- script-injection: every ${{ github.action_path }} in a run: script is now the quoted "$GITHUB_ACTION_PATH". That covers configure_backend.py, package.json (two places), the requirements files, gh_graph.py, agent_tooling.py (two places), pipx and run.py. All these steps use bash.
- unsafe-shell: the backboard installer is now downloaded with curl -o into a mktemp file under $RUNNER_TEMP, checked to be non-empty, then run with sh "$file". It takes no arguments and BACKBOARD_INSTALL is still exported, so it installs to the same place as before. There is no checksum check yet because upstream doesn't publish one.
- github-env-injection: the node spec goes through printf '%s' | tr -d '\n\r' before it is written to GITHUB_OUTPUT. The backboard bin directory is cleaned the same way before it is appended to GITHUB_PATH.
- Note: the edit tool garbled the steps section partway through, probably because its offsets slipped on the multi-byte characters in the file. I rewrote the whole of action.yml: inputs and outputs exactly as upstream had them, and the steps rebuilt from my first clean read with only the fixes above changed. I checked afterwards that no inline action_path or curl pipe is left. Nothing has been run against the rewritten file.

