# Hardening Report: appleboy--ssh-action/v1.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **appleboy--ssh-action/v1.2.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set GitHub Path' step in action.yml writes the value of `${{ github.action_path }}` (a github.* context value) to `$GITHUB_PATH` via an intermediate env var `GITHUB_ACTION_PATH`, without applying the required sanitization (`printf '%s' "$VAR" | tr -d '\n\r'`) before the write. A malicious value containing newline characters in `github.action_path` could inject arbitrary entries into the runner's PATH. The failing pattern is: `run: echo "$GITHUB_ACTION_PATH" >> $GITHUB_PATH` with `env: GITHUB_ACTION_PATH: ${{ github.action_path }}`.

Locations:

- `action.yml:86`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Set GitHub Path' step in action.yml (line 86). The step previously wrote `$GITHUB_ACTION_PATH` (from `${{ github.action_path }}`) directly to `$GITHUB_PATH` without sanitization. The fix adds newline/carriage-return stripping using `safe=$(printf '%s' "$GITHUB_ACTION_PATH" | tr -d '\n\r')` before writing `$safe` to `"$GITHUB_PATH"`, preventing PATH injection via embedded newline characters.

