<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.9.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references two GitHub Actions by mutable version tags instead of pinned 40-character commit SHAs. This exposes the action to supply-chain attacks if the upstream action tags are moved or compromised. Failing references: `actions/setup-python@v7` and `actions/setup-node@v7`.

Locations:

- `action.yml:1`
- `action.yml:1`

### unsafe-shell (severity: high)

The 'Install the coding-agent CLI' step in action.yml pipes remote content directly to a shell interpreter: `curl -fsSL https://app.backboard.io/api/cli | sh`. This pattern executes whatever the remote server returns without any integrity verification, enabling a compromised or MITM'd server to run arbitrary code on the runner.

Locations:

- `action.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ github.action_path }}` expressions inside shell command strings (sub-rule a). Although `github.action_path` is not attacker-controlled, the check requires that NO `${{ ... }}` expression appear inside a `run:` shell command string — all such values must be routed through `env:` variables and then referenced as shell variables. Affected steps and their offending lines include:
- 'Configure the model backend': `run: python ${{ github.action_path }}/src/configure_backend.py`
- 'Resolve the Node line from package.json': `spec="$(python -c ... "${{ github.action_path }}/package.json")"`
- 'Install the coding-agent CLI': `PKG="${{ github.action_path }}/package.json"`
- 'Install pytest + remyxai CLI': `run: pip install -r "${{ github.action_path }}/requirements-action.txt"`
- 'Install pypdf': `run: pip install "$(grep -E '^pypdf' "${{ github.action_path }}/requirements-action-optional.txt")"`
- 'Install gh-graph': `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`
- 'Install cocoindex-code AST search': `pipx install "$(grep -E '^cocoindex-code' "${{ github.action_path }}/requirements-action-optional.txt")"` and `SKILLS_HOME=$(python "${{ github.action_path }}/src/agent_tooling.py" --skills-home)`
- 'Write ENVIRONMENTS.md': `python "${{ github.action_path }}/src/agent_tooling.py"`
- 'Recommend + implement + open PR': `python ${{ github.action_path }}/src/run.py`

Locations:

- `action.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, script-injection

**Notes:**

1. Pinned actions/setup-python@v7 to SHA 5fda3b95a4ea91299a34e894583c3862153e4b97 and actions/setup-node@v7 to SHA 949feb2413d6458794dcd2491c4babbbce0c15c1, preserving the tag as a comment.
2. Fixed unsafe-shell: replaced `curl -fsSL https://app.backboard.io/api/cli | sh` with a two-step download-then-execute pattern using a temp file ($RUNNER_TEMP/backboard_install.sh).
3. Fixed script-injection: moved all 9 occurrences of `${{ github.action_path }}` from run: shell strings into env: blocks as ACTION_PATH, then referenced as $ACTION_PATH in the shell scripts. Affected steps: Configure the model backend, Resolve the Node line from package.json, Install the coding-agent CLI, Install pytest + remyxai CLI, Install pypdf, Install gh-graph, Install cocoindex-code AST search, Write ENVIRONMENTS.md, and Recommend + implement + open PR.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in hardened/action/src/configure_backend.py. The write loop that appends key=value lines to $GITHUB_ENV now sanitizes both keys and values by stripping embedded \n and \r characters (via str.replace) before writing. This prevents an attacker who controls inputs.model or inputs.model-base-url from injecting additional environment variable definitions by embedding newline characters in those values.

