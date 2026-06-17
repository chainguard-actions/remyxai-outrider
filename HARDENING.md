<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.5.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ github.action_path }}` is directly interpolated inside a `run:` shell command string: `python ${{ github.action_path }}/src/run.py`. Any `${{ ... }}` expression inside a `run:` block undergoes YAML template substitution before the shell sees it, making it a script-injection risk. The value should be passed via an `env:` variable and referenced as `$ACTION_PATH` (which is already set automatically by the runner as `$GITHUB_ACTION_PATH`).

Locations:

- `action.yml:222`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable version tags instead of full 40-character SHA commit digests, making the action vulnerable to supply-chain attacks if the upstream action is compromised or the tag is moved:
- `uses: actions/setup-python@v5` (mutable tag `v5`)
- `uses: actions/setup-node@v4` (mutable tag `v4`)
These should be pinned to their full SHA digests, e.g. `actions/setup-python@<40-char-sha> # v5`.

Locations:

- `action.yml:184`
- `action.yml:188`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Three changes made to actions/hardened/remyxai--outrider/v1.5.3/action.yml:
1. Pinned `actions/setup-python@v5` to full SHA `a26af69be951a213d495a4c3e4e4022e16d87065` with `# v5` comment.
2. Pinned `actions/setup-node@v4` to full SHA `49933ea5288caeca8642d1e84afbd3f7d6820020` with `# v4` comment.
3. Fixed script injection on line 222: replaced `python ${{ github.action_path }}/src/run.py` with `python "$GITHUB_ACTION_PATH/src/run.py"`. The `$GITHUB_ACTION_PATH` environment variable is automatically set by the GitHub Actions runner and is equivalent to `github.action_path`, so no `env:` block addition was needed — the value is already safely available as a shell variable without YAML template substitution.

