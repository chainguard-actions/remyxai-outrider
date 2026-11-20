<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.9.0** was hardened automatically. 13 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml composite steps reference actions by mutable version tags instead of full 40-char commit SHAs: 'uses: actions/setup-python@v7' and 'uses: actions/setup-node@v7'.

Locations:

- `action.yml:553`
- `action.yml:584`

### unsafe-shell (severity: high)

The 'Install the coding-agent CLI' step (backboard branch) pipes remote content straight into a shell: 'curl -fsSL https://app.backboard.io/api/cli | sh'. Download the installer to a file, check it (for example against a checksum), then run it as a separate step.

Locations:

- `action.yml:632`

### script-injection (severity: high)

Rule (a): the 'Configure the model backend' step puts an expression directly in a run: block: 'run: python ${{ github.action_path }}/src/configure_backend.py'. Pass it through env (e.g. ACTION_PATH) or use the $GITHUB_ACTION_PATH env var, double-quoted.

Locations:

- `action.yml:571`

### script-injection (severity: high)

Rule (a): the 'Resolve the Node line from package.json' step puts an expression directly in its run: script: '... "${{ github.action_path }}/package.json")"'.

Locations:

- `action.yml:580`

### script-injection (severity: high)

Rule (a): the 'Install the coding-agent CLI' step puts an expression directly in its run: script: 'PKG="${{ github.action_path }}/package.json"'.

Locations:

- `action.yml:597`

### script-injection (severity: high)

Rule (a): the 'Install pytest + remyxai CLI + pyyaml' step puts an expression directly in its run: script: 'run: pip install -r "${{ github.action_path }}/requirements-action.txt"'.

Locations:

- `action.yml:666`

### script-injection (severity: high)

Rule (a): the 'Install pypdf' step puts an expression directly in its run: script: 'run: pip install "$(grep -E '^pypdf' "${{ github.action_path }}/requirements-action-optional.txt")"'.

Locations:

- `action.yml:674`

### script-injection (severity: high)

Rule (a): the 'Install gh-graph selection-pass tool on PATH' step puts an expression directly in its run: script: 'install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph'.

Locations:

- `action.yml:682`

### script-injection (severity: high)

Rule (a): the 'Install cocoindex-code AST search' step puts expressions directly in its run: script: 'pipx install "$(grep -E '^cocoindex-code' "${{ github.action_path }}/requirements-action-optional.txt")"' and 'SKILLS_HOME=$(python "${{ github.action_path }}/src/agent_tooling.py" --skills-home)'.

Locations:

- `action.yml:703`

### script-injection (severity: high)

Rule (a): the 'Write ENVIRONMENTS.md' step puts an expression directly in its run: script: 'python "${{ github.action_path }}/src/agent_tooling.py" --environments-md > ENVIRONMENTS.md'.

Locations:

- `action.yml:741`

### script-injection (severity: high)

Rule (a): the 'Recommend + implement + open PR' step puts an expression directly in its run: script: 'python ${{ github.action_path }}/src/run.py'.

Locations:

- `action.yml:836`

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. Two run: blocks write values derived from inherited or github-context data to special environment files without the `printf '%s' ... | tr -d '\n\r'` sanitization step. (1) 'Install the coding-agent CLI': `dirname "$BB" >> "$GITHUB_PATH"`. $BB is derived from `BACKBOARD_INSTALL="$RUNNER_TEMP/backboard"`, where RUNNER_TEMP is inherited process env, or from `command -v backboard` output. (2) 'Resolve the Node line from package.json': `echo "spec=$spec" >> "$GITHUB_OUTPUT"`. $spec is read from a file located through `${{ github.action_path }}`. Both values are written unsanitized.

Locations:

- `action.yml:667`
- `action.yml:603`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 292874f..6c630be 100644
--- a/action.yml
+++ b/action.yml
@@ -560,8 +560,25 @@ outputs:
 runs:
   using: 'composite'
   steps:
     - name: Set up Python
       uses: actions/setup-python@v7
       with:
         python-version: '3.11'
 
@@ -582,7 +599,7 @@ runs:
         INPUT_PROVIDER: ${{ inputs.provider }}
         INPUT_MODEL: ${{ inputs.model }}
         INPUT_MODEL_BASE_URL: ${{ inputs.model-base-url }}
-      run: python ${{ github.action_path }}/src/configure_backend.py
+      run: python "$GITHUB_ACTION_PATH/src/configure_backend.py"
 
     - name: Resolve the Node line from package.json
       id: node-line
@@ -592,12 +609,13 @@ runs:
       # two cannot drift apart in separate files. The install step below
       # refuses a CLI whose declared engine floor this line does not meet.
       run: |
-        spec="$(python -c 'import json,sys; print(json.load(open(sys.argv[1]))["engines"]["node"])' "${{ github.action_path }}/package.json")"
-        echo "spec=$spec" >> "$GITHUB_OUTPUT"
-        echo "node line: $spec"
+        spec="$(python -c 'import json,sys; print(json.load(open(sys.argv[1]))["engines"]["node"])' "$GITHUB_ACTION_PATH/package.json")"
+        safe_spec="$(printf '%s' "$spec" | tr -d '\n\r')"
+        echo "spec=$safe_spec" >> "$GITHUB_OUTPUT"
+        echo "node line: $safe_spec"
 
     - name: Set up Node (for the agent CLI)
       uses: actions/setup-node@v7
       with:
         node-version: ${{ steps.node-line.outputs.spec }}
 
@@ -612,7 +630,7 @@ runs:
         # Exact versions come from package.json so Dependabot can bump them
         # (#128). A floating `npm install -g <pkg>` would pull whatever
         # shipped overnight into every scheduled run at once.
-        PKG="${{ github.action_path }}/package.json"
+        PKG="$GITHUB_ACTION_PATH/package.json"
         pin() { node -p "require(process.argv[1]).dependencies[process.argv[2]]" "$PKG" "$1"; }
         # npm only *warns* (EBADENGINE) when a package's engines.node is
         # not met and installs anyway; the CLI then works until it does
@@ -643,12 +661,20 @@ runs:
           backboard)
             # Native binary (~99 MB), not an npm package.
             #
-            # BACKBOARD_INSTALL must be *exported*, not prefixed onto the
-            # pipeline: `VAR=x curl ... | sh` sets VAR for curl, not for the
-            # sh that actually runs the installer, so the install silently
-            # lands in $HOME/.backboard and any hardcoded path 127s.
+            # BACKBOARD_INSTALL must be *exported* so the installer script
+            # (run as a separate process below) sees it; otherwise the
+            # install silently lands in $HOME/.backboard.
             export BACKBOARD_INSTALL="$RUNNER_TEMP/backboard"
-            curl -fsSL https://app.backboard.io/api/cli | sh
+            # Download the installer to a file first, then run it, rather
+            # than piping remote content straight into a shell.
+            BB_INSTALLER="$(mktemp "$RUNNER_TEMP/backboard-install.XXXXXX")"
+            curl -fsSL https://app.backboard.io/api/cli -o "$BB_INSTALLER"
+            if [ ! -s "$BB_INSTALLER" ]; then
+              echo "::error::backboard installer download is empty" >&2
+              exit 1
+            fi
+            sh "$BB_INSTALLER"
+            rm -f "$BB_INSTALLER"
             # Resolve wherever it actually landed: the installer also links
             # the binary onto PATH itself, so accept either location rather
             # than depending on one layout staying put.
@@ -658,7 +684,9 @@ runs:
               echo "::error::backboard CLI not found after install" >&2
               exit 1
             fi
-            dirname "$BB" >> "$GITHUB_PATH"
+            BB_DIR="$(dirname "$BB")"
+            BB_DIR_SAFE="$(printf '%s' "$BB_DIR" | tr -d '\n\r')"
+            printf '%s\n' "$BB_DIR_SAFE" >> "$GITHUB_PATH"
             "$BB" --version
             ;;
           *)
@@ -677,7 +705,7 @@ runs:
       # before the agent started (#128). Pins live in requiremen
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, script-injection, github-env-injection, prior-fix

**Notes:**

Pinned actions/setup-python and actions/setup-node to their v7 commit SHAs. In every run: script, replaced ${{ github.action_path }} with "$GITHUB_ACTION_PATH" in double quotes. The backboard installer is now saved to a mktemp file, checked to be non-empty, and then run with sh, with BACKBOARD_INSTALL still exported. Newlines are stripped from the node spec before it goes to GITHUB_OUTPUT and from the backboard directory before it goes to GITHUB_PATH. This repeats the changes from the previous harden. The only remaining @v matches are 'remyxai/outrider@v1' inside description and comment text.

