# Hardening Report: appleboy--ssh-action/v1.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **appleboy--ssh-action/v1.2.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set GitHub Path' step writes the value of $GITHUB_ACTION_PATH (sourced from the attacker-controllable expression `${{ github.action_path }}` via an env: variable) directly to $GITHUB_PATH using `echo "$GITHUB_ACTION_PATH" >> $GITHUB_PATH`, without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`) applied before the write. A newline-injection in this value could allow an attacker to inject arbitrary entries into the runner's PATH.

Locations:

- `action.yml:100`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Set GitHub Path' step in action.yml (line 100) to sanitize the GITHUB_ACTION_PATH value before writing to $GITHUB_PATH. Changed from a direct `echo "$GITHUB_ACTION_PATH" >> $GITHUB_PATH` to a two-step approach: first strip newlines/carriage returns with `safe=$(printf '%s' "$GITHUB_ACTION_PATH" | tr -d '\n\r')`, then write the sanitized value with `echo "$safe" >> "$GITHUB_PATH"`. This prevents newline injection attacks that could allow an attacker to inject arbitrary entries into the runner's PATH.

