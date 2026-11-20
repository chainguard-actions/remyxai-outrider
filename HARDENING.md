<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.9.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.9.2** was hardened automatically. 13 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml composite steps pin actions to version tags, not 40-char commit SHAs: `uses: actions/setup-python@v7` and `uses: actions/setup-node@v7`.

Locations:

- `action.yml:645`
- `action.yml:680`

### script-injection (severity: high)

Rule (a): step 'Configure the model backend' puts an expression straight into run: `run: python ${{ github.action_path }}/src/configure_backend.py`. The expression is also unquoted.

Locations:

- `action.yml:665`

### script-injection (severity: high)

Rule (a): step 'Resolve the Node line from package.json' puts an expression into run: `... "${{ github.action_path }}/package.json")"`.

Locations:

- `action.yml:674`

### script-injection (severity: high)

Rule (a): step 'Install the coding-agent CLI' puts an expression into run: `PKG="${{ github.action_path }}/package.json"`.

Locations:

- `action.yml:693`

### script-injection (severity: high)

Rule (a): step 'Install pytest + remyxai CLI + pyyaml' puts an expression into run: `pip install -r "${{ github.action_path }}/requirements-action.txt"`.

Locations:

- `action.yml:755`

### script-injection (severity: high)

Rule (a): step 'Install pypdf' puts an expression into run: `pip install "$(grep -E '^pypdf' "${{ github.action_path }}/requirements-action-optional.txt")"`.

Locations:

- `action.yml:763`

### script-injection (severity: high)

Rule (a): step 'Install gh-graph selection-pass tool on PATH' puts an expression into run: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`.

Locations:

- `action.yml:771`

### script-injection (severity: high)

Rule (a): step 'Install cocoindex-code AST search' puts expressions into run: `pipx install "$(grep -E '^cocoindex-code' "${{ github.action_path }}/requirements-action-optional.txt")"` and `SKILLS_HOME=$(python "${{ github.action_path }}/src/agent_tooling.py" --skills-home)`.

Locations:

- `action.yml:793`

### script-injection (severity: high)

Rule (a): step 'Write ENVIRONMENTS.md...' puts an expression into run: `python "${{ github.action_path }}/src/agent_tooling.py" \`.

Locations:

- `action.yml:830`

### script-injection (severity: high)

Rule (a): step 'Recommend + implement + open PR' puts an expression into run: `python ${{ github.action_path }}/src/run.py`. The expression is also unquoted.

Locations:

- `action.yml:905`

### unsafe-shell (severity: high)

Step 'Install the coding-agent CLI' (backboard branch) pipes remote content straight into a shell: `curl -fsSL https://app.backboard.io/api/cli | sh`. Download the installer to a file, check it, then run it as a separate step.

Locations:

- `action.yml:724`

### github-env-injection (severity: high)

Step 'Install the coding-agent CLI' writes `dirname "$BB" >> "$GITHUB_PATH"` with no `printf '%s' ... | tr -d '\n\r'` step first. `$BB` comes from the inherited environment (`$RUNNER_TEMP` via BACKBOARD_INSTALL) or from `command -v backboard`. Neither is a literal computed in the block, so the value is not stripped of newlines before the PATH write.

Locations:

- `action.yml:733`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 1035809..efa0f8c 100644
--- a/action.yml
+++ b/action.yml
@@ -630,8 +630,25 @@ outputs:
 runs:
   using: 'composite'
   steps:
     - name: Set up Python
       uses: actions/setup-python@v7
       with:
         python-version: ${{ inputs.python-version }}
 
@@ -652,7 +669,7 @@ runs:
         INPUT_PROVIDER: ${{ inputs.provider }}
         INPUT_MODEL: ${{ inputs.model }}
         INPUT_MODEL_BASE_URL: ${{ inputs.model-base-url }}
-      run: python ${{ github.action_path }}/src/configure_backend.py
+      run: python "$GITHUB_ACTION_PATH/src/configure_backend.py"
 
     - name: Resolve the Node line from package.json
       id: node-line
@@ -662,12 +679,13 @@ runs:
       # two cannot drift apart in separate files. The install step below
       # refuses a CLI whose declared engine floor this line does not meet.
       run: |
-        spec="$(python -c 'import json,sys; print(json.load(open(sys.argv[1]))["engines"]["node"])' "${{ github.action_path }}/package.json")"
-        echo "spec=$spec" >> "$GITHUB_OUTPUT"
+        spec="$(python -c 'import json,sys; print(json.load(open(sys.argv[1]))["engines"]["node"])' "$GITHUB_ACTION_PATH/package.json")"
+        safe_spec="$(printf '%s' "$spec" | tr -d '\n\r')"
+        echo "spec=$safe_spec" >> "$GITHUB_OUTPUT"
         echo "node line: $spec"
 
     - name: Set up Node (for the agent CLI)
       uses: actions/setup-node@v7
       with:
         node-version: ${{ steps.node-line.outputs.spec }}
 
@@ -682,7 +700,7 @@ runs:
         # Exact versions come from package.json so Dependabot can bump them
         # (#128). A floating `npm install -g <pkg>` would pull whatever
         # shipped overnight into every scheduled run at once.
-        PKG="${{ github.action_path }}/package.json"
+        PKG="$GITHUB_ACTION_PATH/package.json"
         pin() { node -p "require(process.argv[1]).dependencies[process.argv[2]]" "$PKG" "$1"; }
         # npm only *warns* (EBADENGINE) when a package's engines.node is
         # not met and installs anyway; the CLI then works until it does
@@ -718,7 +736,14 @@ runs:
             # sh that actually runs the installer, so the install silently
             # lands in $HOME/.backboard and any hardcoded path 127s.
             export BACKBOARD_INSTALL="$RUNNER_TEMP/backboard"
-            curl -fsSL https://app.backboard.io/api/cli | sh
+            BB_INSTALLER="$(mktemp "$RUNNER_TEMP/backboard-install.XXXXXX.sh")"
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
@@ -728,7 +753,8 @@ runs:
               echo "::error::backboard CLI not found after install" >&2
               exit 1
             fi
-            dirname "$BB" >> "$GITHUB_PATH"
+            BB_DIR="$(dirname "$BB")"
+            printf '%s\n' "$(printf '%s' "$BB_DIR" | tr -d '\n\r')" >> "$GITHUB_PATH"
             "$BB" --version
             ;;
           *)
@@ -747,7 +773,7 @@ runs:
       # before the agent started (#128). Pins live in requirements-action.txt
       # so Dependabot can bump them; tests/test_action_install_surface.py
       # enforces both rules.
-      run: pip install -r "${{ github.action_path }}/requirements-action.txt"
+      run: pip install -r "$GITHUB_ACTION_PATH/requirements-action.txt"
 
     - name: Install pypdf (optional, arxiv PDF license retry)
       shell: bash
@@ -756,7 +782,7 @@ runs:
       # must not fail the run. Steps that only improve the agent's inputs
       # are best-effort by rule; the same test checks this flag is set.
       continue-on-error: true
-      run: pip install "$(grep -E '^pypdf' "${{ github.action_path }}/requirements-action
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, unsafe-shell, github-env-injection, prior-fix

**Notes:**

Pinned actions/setup-python (6d4fd22b… # v7) and actions/setup-node (949feb24… # v7) to commit SHAs. Replaced every ${{ github.action_path }} in run: scripts with a quoted "$GITHUB_ACTION_PATH". The backboard installer is now downloaded to a mktemp file, checked to be non-empty, then run with sh with BACKBOARD_INSTALL still exported. Newlines are stripped from the backboard dir before it is written to GITHUB_PATH. Re-applied the prior fix that strips newlines from the node spec output. The remaining 'uses: remyxai/outrider@v1' matches are only in comments and descriptions.

