<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.5.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references actions/setup-python@v5 and actions/setup-node@v4 using mutable version tags instead of pinned 40-character commit SHAs. If the upstream action is compromised or the tag is moved, the action will silently execute different code. The workflow file outrider.yml also references actions/checkout@v4 using a mutable tag.

Locations:

- `action.yml:218`
- `action.yml:222`
- `.github/workflows/outrider.yml:19`

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command string. In action.yml, the final run: step executes `python ${{ github.action_path }}/src/run.py` — the github.action_path context value flows through YAML template substitution before the shell sees it, making it a script-injection risk. In .github/workflows/outrider.yml, the 'Mint Remyx bot token' run: step directly interpolates ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} inside the shell command string passed to curl.

Locations:

- `action.yml:253`
- `.github/workflows/outrider.yml:26`

### github-env-injection (severity: high)

In .github/workflows/outrider.yml, the 'Mint Remyx bot token' step writes `echo "token=$token" >> "$GITHUB_OUTPUT"` where $token is derived from a curl call that embeds ${{ github.repository }} (an attacker-controllable value) in the request body. The token value written to GITHUB_OUTPUT is not sanitized with `printf '%s' ... | tr -d '\n\r'` before the write, allowing a malicious repository name containing newlines to inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/outrider.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings:

1. unpinned-uses: Pinned actions/setup-python@v5 → SHA a26af69..., actions/setup-node@v4 → SHA 49933ea..., and actions/checkout@v4 → SHA 11d5960... with inline tag comments.

2. script-injection (action.yml): Moved ${{ github.action_path }} from the run: shell string into an env: variable ACTION_PATH; shell now uses python "$ACTION_PATH/src/run.py".

3. script-injection + github-env-injection (outrider.yml): Moved ${{ secrets.REMYX_API_KEY }} and ${{ github.repository }} into step-level env: variables; curl now uses $REMYX_API_KEY and $GITHUB_REPOSITORY. The token written to GITHUB_OUTPUT is sanitized via `printf '%s' "$token" | tr -d '\n\r'` to prevent newline injection.

