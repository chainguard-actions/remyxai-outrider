<!-- markdownlint-disable -->

# Hardening Report: remyxai--outrider/v1.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remyxai--outrider/v1.9.1** was hardened automatically. 13 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Composite action steps reference actions by mutable tag rather than a full 40-char commit SHA: `uses: actions/setup-python@v7` and `uses: actions/setup-node@v7`.

Locations:

- `action.yml:624`
- `action.yml:660`

### unsafe-shell (severity: high)

Step 'Install the coding-agent CLI' (backboard branch) pipes remote content directly to a shell: `curl -fsSL https://app.backboard.io/api/cli | sh`. Download the installer to a file, verify it, then execute it separately.

Locations:

- `action.yml:711`

### script-injection (severity: high)

Rule (a): step 'Configure the model backend' interpolates an expression directly in run: `run: python ${{ github.action_path }}/src/configure_backend.py`.

Locations:

- `action.yml:645`

### script-injection (severity: high)

Rule (a): step 'Resolve the Node line from package.json' interpolates `"${{ github.action_path }}/package.json"` directly inside the run: script.

Locations:

- `action.yml:655`

### script-injection (severity: high)

Rule (a): step 'Install the coding-agent CLI' interpolates an expression directly in run: `PKG="${{ github.action_path }}/package.json"`.

Locations:

- `action.yml:675`

### script-injection (severity: high)

Rule (a): step 'Install pytest + remyxai CLI + pyyaml' interpolates an expression directly in run: `pip install -r "${{ github.action_path }}/requirements-action.txt"`.

Locations:

- `action.yml:740`

### script-injection (severity: high)

Rule (a): step 'Install pypdf' interpolates an expression directly in run: `pip install "$(grep -E '^pypdf' "${{ github.action_path }}/requirements-action-optional.txt")"`.

Locations:

- `action.yml:749`

### script-injection (severity: high)

Rule (a): step 'Install gh-graph selection-pass tool on PATH' interpolates an expression directly in run: `install -m 0755 "${{ github.action_path }}/src/gh_graph.py" /usr/local/bin/gh-graph`.

Locations:

- `action.yml:758`

### script-injection (severity: high)

Rule (a): step 'Install cocoindex-code AST search' interpolates `${{ github.action_path }}` directly in run: (`pipx install "$(grep -E '^cocoindex-code' "${{ github.action_path }}/requirements-action-optional.txt")"` and `SKILLS_HOME=$(python "${{ github.action_path }}/src/agent_tooling.py" --skills-home)`).

Locations:

- `action.yml:782`

### script-injection (severity: high)

Rule (a): step 'Write ENVIRONMENTS.md' interpolates an expression directly in run: `python "${{ github.action_path }}/src/agent_tooling.py" \`.

Locations:

- `action.yml:825`

### script-injection (severity: high)

Rule (a): step 'Recommend + implement + open PR' interpolates an expression directly in run: `python ${{ github.action_path }}/src/run.py`.

Locations:

- `action.yml:906`

### github-env-injection (severity: high)

Step 'Install the coding-agent CLI' writes `dirname "$BB" >> "$GITHUB_PATH"` without the sanitization step. BB comes from BACKBOARD_INSTALL="$RUNNER_TEMP/backboard" (RUNNER_TEMP is an inherited process env var) or from `command -v backboard`. The script does not run `printf '%s' ... | tr -d '\n\r'` before the write.

Locations:

- `action.yml:721`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 75baf02..83ed808 100644
--- a/action.yml
+++ b/action.yml
@@ -620,8 +620,25 @@ outputs:
 runs:
   using: 'composite'
   steps:
     - name: Set up Python
       uses: actions/setup-python@v7
       with:
         python-version: '3.11'
 
@@ -642,7 +659,7 @@ runs:
         INPUT_PROVIDER: ${{ inputs.provider }}
         INPUT_MODEL: ${{ inputs.model }}
         INPUT_MODEL_BASE_URL: ${{ inputs.model-base-url }}
-      run: python ${{ github.action_path }}/src/configure_backend.py
+      run: python "$GITHUB_ACTION_PATH/src/configure_backend.py"
 
     - name: Resolve the Node line from package.json
       id: node-line
@@ -652,12 +669,12 @@ runs:
       # two cannot drift apart in separate files. The install step below
       # refuses a CLI whose declared engine floor this line does not meet.
       run: |
-        spec="$(python -c 'import json,sys; print(json.load(open(sys.argv[1]))["engines"]["node"])' "${{ github.action_path }}/package.json")"
+        spec="$(python -c 'import json,sys; print(json.load(open(sys.argv[1]))["engines"]["node"])' "$GITHUB_ACTION_PATH/package.json")"
         echo "spec=$spec" >> "$GITHUB_OUTPUT"
         echo "node line: $spec"
 
     - name: Set up Node (for the agent CLI)
       uses: actions/setup-node@v7
       with:
         node-version: ${{ steps.node-line.outputs.spec }}
 
@@ -672,7 +689,7 @@ runs:
         # Exact versions come from package.json so Dependabot can bump them
         # (#128). A floating `npm install -g <pkg>` would pull whatever
         # shipped overnight into every scheduled run at once.
-        PKG="${{ github.action_path }}/package.json"
+        PKG="$GITHUB_ACTION_PATH/package.json"
         pin() { node -p "require(process.argv[1]).dependencies[process.argv[2]]" "$PKG" "$1"; }
         # npm only *warns* (EBADENGINE) when a package's engines.node is
         # not met and installs anyway; the CLI then works until it does
@@ -708,7 +725,14 @@ runs:
             # sh that actually runs the installer, so the install silently
             # lands in $HOME/.backboard and any hardcoded path 127s.
             export BACKBOARD_INSTALL="$RUNNER_TEMP/backboard"
-            curl -fsSL https://app.backboard.io/api/cli | sh
+            BB_INSTALLER="$(mktemp)"
+            curl -fsSL https://app.backboard.io/api/cli -o "$BB_INSTALLER"
+            if [ ! -s "$BB_INSTALLER" ]; then
+              echo "::error::failed to download the backboard installer" >&2
+              exit 1
+            fi
+            sh "$BB_INSTALLER"
+            rm -f "$BB_INSTALLER"
             # Resolve wherever it actually landed: the installer also links
             # the binary onto PATH itself, so accept either location rather
             # than depending on one layout staying put.
@@ -718,7 +742,8 @@ runs:
               echo "::error::backboard CLI not found after install" >&2
               exit 1
             fi
-            dirname "$BB" >> "$GITHUB_PATH"
+            BB_DIR="$(dirname "$BB")"
+            printf '%s\n' "$(printf '%s' "$BB_DIR" | tr -d '\n\r')" >> "$GITHUB_PATH"
             "$BB" --version
             ;;
           *)
@@ -737,7 +762,7 @@ runs:
       # before the agent started (#128). Pins live in requirements-action.txt
       # so Dependabot can bump them; tests/test_action_install_surface.py
       # enforces both rules.
-      run: pip install -r "${{ github.action_path }}/requirements-action.txt"
+      run: pip install -r "$GITHUB_ACTION_PATH/requirements-action.txt"
 
     - name: Install pypdf (optional, arxiv PDF license retry)
       shell: bash
@@ -746,7 +771,7 @@ runs:
       # must not fail the run. Steps that only improve the agent's inputs
       # are best-effort by rule; the same test checks this flag is set.
       continue-on-error: true
-      run: pip install "$(grep -E '^pypdf' "${{ github.action_path }}/requirements-action-optional.txt")"
+      run: pip install "$(grep -E '^pypdf' "$GITHUB_ACTION_PATH/requirements-action-optional.txt")"
 
     - name: Install gh-graph selection-pass tool o
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, script-injection, github-env-injection, prior-fix

**Notes:**

Fixed all findings in action.yml, matching the previous harden's patch:
- unpinned-uses: pinned actions/setup-python to 6d4fd22b20465ccc0f7354476588e8df7c9ca0e0 and actions/setup-node to 949feb2413d6458794dcd2491c4babbbce0c15c1. Both SHAs were looked up, not guessed, and each line keeps a '# v7' comment.
- unsafe-shell: the backboard installer is now downloaded to a mktemp file and checked for being non-empty before it runs with 'sh "$BB_INSTALLER"'. The file is deleted afterwards. BACKBOARD_INSTALL is still exported, so the installer still sees it.
- script-injection: every ${{ github.action_path }} inside a run: script now uses the quoted "$GITHUB_ACTION_PATH" variable instead. This covers configure_backend.py, the package.json reads, the requirements files, gh_graph.py, agent_tooling.py and run.py. All of these steps run in bash.
- github-env-injection: the backboard directory is now stripped of newlines with tr -d '\n\r' before it is written to $GITHUB_PATH.
- prior-fix: re-applied every change the previous patch showed. That patch was cut off in the request, and it also added about 17 lines at the start of runs.steps that it didn't show. I couldn't recover those lines and didn't guess at them. A search afterwards found no remaining github.action_path expressions, tag-pinned uses: or pipe-to-shell commands, apart from mentions in comments.

