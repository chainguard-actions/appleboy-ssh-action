<!-- markdownlint-disable -->

# Hardening Report: appleboy--ssh-action/v1.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **appleboy--ssh-action/v1.2.5** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or version strings instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks. Failing references include: goreleaser.yml: actions/checkout@v4, actions/setup-go@v5, goreleaser/goreleaser-action@v6; main.yml: actions/checkout@v4 (multiple), fscarmen/warp-on-actions@v1.1; stable.yml: actions/checkout@v4 (multiple), appleboy/ssh-action@v1 (multiple); trivy-scan.yml: actions/checkout@v4, aquasecurity/trivy-action@0.33.1 (twice), github/codeql-action/upload-sarif@v3.

Locations:

- `.github/workflows/goreleaser.yml:14`
- `.github/workflows/main.yml:8`
- `.github/workflows/stable.yml:8`
- `.github/workflows/trivy-scan.yml:18`

### missing-permissions (severity: medium)

main.yml and stable.yml have no top-level `permissions:` key and no job-level `permissions:` block on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/stable.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in main.yml directly interpolate `${{ steps.*.outputs.stdout }}` expressions inside shell commands (sub-rule a). The stdout output of a remote SSH command is attacker-influenced and is embedded verbatim into shell strings such as `echo "stdout: ${{ steps.stdout.outputs.stdout }}"`, `if ! echo "${{ steps.stdout-multiline.outputs.stdout }}" | grep -q ...`, and `OUTPUT="${{ steps.stdout-multiline.outputs.stdout }}"`/`${{ steps.stdout-with-special-chars.outputs.stdout }}`. An attacker controlling the remote SSH output could inject arbitrary shell commands.

Locations:

- `.github/workflows/main.yml:368`
- `.github/workflows/main.yml:390`
- `.github/workflows/main.yml:430`
- `.github/workflows/main.yml:466`
- `.github/workflows/main.yml:490`

### github-env-injection (severity: high)

In action.yml, the 'Set GitHub Path' step writes the env var `$GITHUB_ACTION_PATH` (sourced from `${{ github.action_path }}`) directly to `$GITHUB_PATH` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The command `echo "$GITHUB_ACTION_PATH" >> $GITHUB_PATH` allows a newline-containing value to inject additional entries into the PATH, enabling path-hijacking attacks.

Locations:

- `action.yml:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:
1. unpinned-uses: Pinned all action references to full SHA digests: actions/checkout@11d5960a326750d5838078e36cf38b85af677262 (v4), actions/setup-go@40f1582b2485089dde7abd97c1529aa768e1baff (v5), goreleaser/goreleaser-action@e435ccd777264be153ace6237001ef4d979d3a7a (v6), fscarmen/warp-on-actions@e8640954f3b22c9e96e63f9ea9cae5df032f40d3 (v1.1), appleboy/ssh-action@0ff4204d59e8e51228ff73bce53f80d53301dee2 (v1), aquasecurity/trivy-action@b6643a29fecd7f34b3597bc6acb0a98b03d33ff8 (v0.33.1), github/codeql-action/upload-sarif@f3712979fa5f215279b101dd0a2e3bdfb4353324 (v3).
2. missing-permissions: Added `permissions: {}` top-level block to main.yml and stable.yml.
3. script-injection: Moved all ${{ steps.*.outputs.stdout }} expressions from run: blocks into env: blocks (STEP_STDOUT) in main.yml for 5 steps: check stdout, check multiline output, check special characters output, check stdout 01, check stdout 02.
4. github-env-injection: Fixed action.yml 'Set GitHub Path' step to sanitize GITHUB_ACTION_PATH with `printf '%s' "$GITHUB_ACTION_PATH" | tr -d '\n\r'` before writing to $GITHUB_PATH.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in both .github/workflows/main.yml and .github/workflows/stable.yml. In main.yml: fixed 6 occurrences of `-e PUBLIC_KEY="${{ env.PUBLIC_KEY }}"` in check-ssh-key, support-key-passphrase, multiple-server (2 docker runs), support-ed25519-key, and testing-with-env jobs, plus 1 occurrence of `-e USER_PASSWORD='${{ env.PASS }}'` in testing07 job. In stable.yml: fixed 6 occurrences of `-e PUBLIC_KEY="${{ env.PUBLIC_KEY }}"` in check-ssh-key, support-key-passphrase, multiple-server (2 docker runs), support-ed25519-key, and testing-with-env jobs. Each fix moves the `${{ }}` expression into the step's `env:` block and replaces the shell interpolation with a plain `$VAR` reference, preventing shell metacharacters in the value from being interpreted by the shell.

