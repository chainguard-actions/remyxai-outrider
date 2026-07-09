<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.7.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **remyxai--outrider/v1.7.11** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses actions/setup-python@v5 and actions/setup-node@v4 — both are tag references, not pinned to a full 40-character SHA commit hash. This exposes the action to supply-chain attacks if the upstream action tag is moved or compromised. Similarly, .github/workflows/outrider.yml uses actions/checkout@v4 (tag, not SHA).

Locations:

- `action.yml:311`
- `action.yml:316`
- `.github/workflows/outrider.yml:22`

### script-injection (severity: high)

Sub-rule (a): ${{ ... }} expressions are interpolated directly inside run: shell command strings. (1) action.yml 'Install gh-graph' step: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py"` — github.action_path is injected directly into the shell command. (2) action.yml 'Recommend + implement + open PR' step: `python ${{ github.action_path }}/src/run.py` — same issue. (3) .github/workflows/outrider.yml 'Mint Remyx bot token' step: `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` are interpolated directly into the curl command string inside the run: block.

Locations:

- `action.yml:330`
- `action.yml:490`
- `.github/workflows/outrider.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all findings:
1. unpinned-uses: Pinned actions/setup-python@v5 → SHA a26af69be951a213d495a4c3e4e4022e16d87065, actions/setup-node@v4 → SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, and actions/checkout@v4 → SHA 34e114876b0b11c390a56381ad16ebd13914f8d5. All retain the original tag as a comment.
2. script-injection: (a) action.yml 'Install gh-graph' step: moved github.action_path into env var ACTION_PATH, referenced as $ACTION_PATH in shell. (b) action.yml 'Recommend + implement + open PR' step: added ACTION_PATH to the existing env block (avoiding duplicate env: keys), referenced as "$ACTION_PATH" in the run script. (c) outrider.yml 'Mint Remyx bot token' step: moved secrets.REMYX_API_KEY and github.repository into step-level env vars and referenced them as plain env vars in the curl command.

