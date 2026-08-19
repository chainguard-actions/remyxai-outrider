<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.5.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references `actions/setup-python@v5` and `actions/setup-node@v4` using mutable version tags instead of pinned 40-character commit SHAs. If these actions are compromised or the tags are moved, the action will silently execute attacker-controlled code. .github/workflows/outrider.yml similarly references `actions/checkout@v4` using a mutable tag.

Locations:

- `action.yml:148`
- `action.yml:153`
- `.github/workflows/outrider.yml:17`

### script-injection (severity: high)

Sub-rule (a): `${{ }}` expressions are interpolated directly inside `run:` shell command strings, bypassing shell quoting and enabling injection. (1) action.yml final run block: `python ${{ github.action_path }}/src/run.py` — the `github.action_path` context is expanded by the template engine before the shell sees it. (2) outrider.yml 'Mint Remyx bot token' run block: `${{ secrets.REMYX_API_KEY }}` and `${{ github.repository }}` are interpolated directly into the curl command string — `github.repository` is attacker-influenced (e.g. via a fork with a crafted repo name) and both values flow through YAML template substitution before the shell parses them.

Locations:

- `action.yml:196`
- `.github/workflows/outrider.yml:26`

### github-env-injection (severity: high)

outrider.yml 'Mint Remyx bot token' step writes `echo "token=$token" >> "$GITHUB_OUTPUT"` where `$token` is the response from a curl call whose request body embeds `${{ github.repository }}` (a workflow-controlled value). The value is written to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' "$token" | tr -d '\n\r'`). A crafted repository name or a malicious API response containing newline characters could inject additional key=value pairs into GITHUB_OUTPUT, poisoning downstream step outputs.

Locations:

- `.github/workflows/outrider.yml:28`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings:

1. unpinned-uses: Pinned actions/setup-python@v5 to SHA a26af69be951a213d495a4c3e4e4022e16d87065, actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020 (both in action.yml), and actions/checkout@v4 to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 (in outrider.yml). All retain the original tag in a trailing comment.

2. script-injection: (a) In action.yml, moved ${{ github.action_path }} into an env var ACTION_PATH and referenced it as "$ACTION_PATH/src/run.py" in the run block. (b) In outrider.yml, moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} into step-level env vars (REMYX_API_KEY and GH_REPOSITORY) and referenced them as plain shell variables in the curl command.

3. github-env-injection: In outrider.yml, the minted token is now sanitized with `printf '%s' "$token" | tr -d '\n\r'` before being written to $GITHUB_OUTPUT, preventing newline-based output injection attacks.

