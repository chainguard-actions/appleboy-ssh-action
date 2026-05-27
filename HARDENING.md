# Hardening Report: appleboy--ssh-action/v1.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **appleboy--ssh-action/v1.2.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set GitHub Path' step writes the value of $GITHUB_ACTION_PATH (sourced from the attacker-influenced expression `${{ github.action_path }}` via an env: variable) directly to $GITHUB_PATH without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). Per the check specification, all `github.*` context values are considered attacker-controlled. A malicious value containing newline characters could inject arbitrary entries into $GITHUB_PATH, potentially hijacking subsequent command lookups. The safe pattern requires sanitizing the value before writing: `safe=$(printf '%s' "$GITHUB_ACTION_PATH" | tr -d '\n\r'); echo "$safe" >> $GITHUB_PATH`.

Locations:

- `action.yml:98`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Set GitHub Path' step in action.yml to sanitize the GITHUB_ACTION_PATH value before writing to $GITHUB_PATH. Changed the single-line `echo "$GITHUB_ACTION_PATH" >> $GITHUB_PATH` to a multi-line script that first strips newline/carriage return characters using `safe=$(printf '%s' "$GITHUB_ACTION_PATH" | tr -d '\n\r')` and then writes the sanitized value with `echo "$safe" >> $GITHUB_PATH`. This prevents an attacker from injecting arbitrary entries into $GITHUB_PATH via newline characters in the github.action_path context value.

