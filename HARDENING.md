<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.8.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two composite action steps use mutable tag refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved. Failing references: `actions/setup-python@v5` and `actions/setup-node@v4`.

Locations:

- `action.yml:436`
- `action.yml:444`

### unsafe-shell (severity: high)

The 'Install the coding-agent CLI' step pipes a remote script directly to a shell interpreter without first downloading and inspecting it: `curl -fsSL https://app.backboard.io/api/cli | sh`. This allows the remote server to execute arbitrary code on the runner.

Locations:

- `action.yml:468`

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks directly interpolate `${{ github.action_path }}` inside shell command strings. Any `${{ ... }}` expression inside a `run:` script is a script-injection risk because the value is substituted by the template engine before the shell parses the command. Affected lines include: `run: python ${{ github.action_path }}/src/configure_backend.py`, `install -m 0755 "${{ github.action_path }}/src/gh_graph.py"`, `SKILLS_HOME=$(python "${{ github.action_path }}/src/agent_tooling.py" --skills-home)`, `python "${{ github.action_path }}/src/agent_tooling.py" --environments-md`, and `python ${{ github.action_path }}/src/run.py`.

Locations:

- `action.yml:449`
- `action.yml:490`
- `action.yml:502`
- `action.yml:519`
- `action.yml:536`

### github-env-injection (severity: high)

The 'Configure the model backend' step sets env vars `INPUT_MODEL=${{ inputs.model }}` and `INPUT_MODEL_BASE_URL=${{ inputs.model-base-url }}` and then invokes `src/configure_backend.py`, which writes these caller-supplied values directly to `$GITHUB_ENV` (via `routing.env` populated from `passthrough_env(model=model, base_url=base_url_override)`) without any newline sanitization (`printf '%s' ... | tr -d '\n\r'`). An attacker who controls the `model` or `model-base-url` inputs can inject arbitrary environment variables into subsequent steps by embedding newlines in the value.

Locations:

- `action.yml:438`
- `src/configure_backend.py:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, script-injection, github-env-injection

**Notes:**

1. unpinned-uses: Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065 and actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, both with tag comments.
2. unsafe-shell: Replaced `curl -fsSL https://app.backboard.io/api/cli | sh` with a download-then-execute pattern using a mktemp file, followed by cleanup.
3. script-injection: Moved all 5 occurrences of `${{ github.action_path }}` from run: blocks into env: blocks as ACTION_PATH, then referenced as $ACTION_PATH in the shell scripts. Affected steps: 'Configure the model backend', 'Install gh-graph selection-pass tool on PATH', 'Install cocoindex-code AST search', 'Write ENVIRONMENTS.md', and 'Recommend + implement + open PR'.
4. github-env-injection: In src/configure_backend.py, added newline/carriage-return sanitization (stripping \n and \r) from all values before writing them to $GITHUB_ENV, preventing injection via caller-supplied model or model-base-url inputs.

