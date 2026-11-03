<!-- markdownlint-disable -->

# Hardening Report: appleboy--ssh-action/v1.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **appleboy--ssh-action/v1.2.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set GitHub Path' step runs `echo "$GITHUB_ACTION_PATH" >> $GITHUB_PATH`, where GITHUB_ACTION_PATH is set from `${{ github.action_path }}` (a github.* context). The value is written to $GITHUB_PATH without first passing it through `printf '%s' ... | tr -d '\n\r'` (case c/d).

Locations:

- `action.yml:97`

### github-env-injection (severity: high)

The entrypoint step (run: entrypoint.sh) writes multi-line command output to $GITHUB_OUTPUT between a fixed heredoc delimiter: `echo 'stdout<<EOF' >> "${GITHUB_OUTPUT}"`, then `"${TARGET}" "$@" | tee -a "${GITHUB_OUTPUT}"`, then `echo 'EOF' >> "${GITHUB_OUTPUT}"`. The output comes from remote commands built from inputs.script/inputs.envs. If any line of it equals 'EOF', the block ends early and the remaining lines are added as extra outputs (case f). The delimiter needs to be generated randomly, for example from /dev/urandom.

Locations:

- `entrypoint.sh:75`
- `action.yml:103`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 7027751..b76f9bc 100644
--- a/action.yml
+++ b/action.yml
@@ -84,17 +84,34 @@ inputs:
   version:
     description: |
       The version of drone-ssh to use.
-
 outputs:
   stdout:
     description: "Standard output of the executed commands when capture_stdout is enabled."
     value: ${{ steps.entrypoint.outputs.stdout }}
-
 runs:
   using: "composite"
   steps:
     - name: Set GitHub Path
-      run: echo "$GITHUB_ACTION_PATH" >> $GITHUB_PATH
+      run: |
+        safe=$(printf '%s' "$GITHUB_ACTION_PATH" | tr -d '\n\r')
+        echo "$safe" >> "$GITHUB_PATH"
       shell: bash
       env:
         GITHUB_ACTION_PATH: ${{ github.action_path }}
@@ -139,7 +156,6 @@ runs:
         INPUT_CAPTURE_STDOUT: ${{ inputs.capture_stdout }}
         INPUT_CURL_INSECURE: ${{ inputs.curl_insecure }}
         DRONE_SSH_VERSION: ${{ inputs.version }}
-
 branding:
   icon: "terminal"
   color: "gray-dark"

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, prior-fix

**Notes:**

action.yml: the 'Set GitHub Path' step now removes newlines from GITHUB_ACTION_PATH with printf | tr -d '\n\r' before writing it to the quoted "$GITHUB_PATH". This re-applies the previous harden's fix. entrypoint.sh: the fixed 'EOF' that marked the end of the stdout output is now a random marker made fresh on each run from /dev/urandom (EOF_<32 hex chars>). The output lines are still written as they are.

