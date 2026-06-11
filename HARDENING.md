<!-- markdownlint-disable -->

# Hardening Report: appleboy--ssh-action/v1.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **appleboy--ssh-action/v1.2.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set GitHub Path' step in action.yml sets the env var GITHUB_ACTION_PATH from `${{ github.action_path }}` (a workflow-controlled value) and then writes it directly to $GITHUB_PATH via `echo "$GITHUB_ACTION_PATH" >> $GITHUB_PATH` without the required sanitization step (`printf '%s' "$GITHUB_ACTION_PATH" | tr -d '\n\r'`). A calling workflow could supply a malicious value containing newlines to inject arbitrary entries into $GITHUB_PATH.

Locations:

- `action.yml:86`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Set GitHub Path' step in action.yml (line 86) by sanitizing the GITHUB_ACTION_PATH value before writing to $GITHUB_PATH. Changed the run script from a direct `echo "$GITHUB_ACTION_PATH" >> $GITHUB_PATH` to a multi-line script that first strips newlines/carriage returns using `printf '%s' "$GITHUB_ACTION_PATH" | tr -d '\n\r'`, stores the result in `safe`, then writes `$safe` to `"$GITHUB_PATH"`. This prevents newline injection attacks where a malicious value could inject arbitrary entries into $GITHUB_PATH.

