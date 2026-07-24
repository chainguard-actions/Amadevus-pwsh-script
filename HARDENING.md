<!-- markdownlint-disable -->

# Hardening Report: Amadevus--pwsh-script/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Amadevus--pwsh-script/v2.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command. In action.yml, the run: block uses `run: ${{ github.action_path }}/action.ps1`, which injects the github.action_path context value directly into the shell command string before the shell ever sees it. Any ${{ }} expression in a run: block is a script-injection risk regardless of which context it reads from.

Locations:

- `action.yml:19`

### script-injection (severity: high)

Sub-rule (a): Multiple run: blocks in ci.yml directly interpolate ${{ steps.*.outputs.* }} and ${{ steps.*.outcome }} expressions into shell command strings. Specifically: `$result = '${{ steps.test-result-string.outputs.result }}'` (line ~57), `$result = '${{ steps.test-result-object.outputs.result }}'` (line ~65), `$outcome = '${{ steps.test-throwing-fails-action.outcome }}'` (line ~74), `$errMsg = '${{ steps.test-throwing-fails-action.outputs.error }}'` (line ~77), and `$result = '${{ steps.Set-ActionFailed.outcome }}'` (line ~175). These step outputs could contain attacker-controlled content (e.g., from the action's script input) and are interpolated directly into the shell before quoting.

Locations:

- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:65`
- `.github/workflows/ci.yml:74`
- `.github/workflows/ci.yml:77`
- `.github/workflows/ci.yml:175`

### unpinned-uses (severity: high)

All uses: references in workflow files use mutable tag or version refs instead of full 40-character SHA commit digests, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references: chatops.yml: `peter-evans/slash-command-dispatch@v1`; ci.yml: `actions/checkout@v2` (×2); demo-command.yml: `peter-evans/create-or-update-comment@v1` (×4), `Amadevus/pwsh-script@v1` (×3); tag-command.yml: `peter-evans/create-or-update-comment@v1` (×3), `actions/checkout@v2`.

Locations:

- `.github/workflows/chatops.yml:10`
- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:40`
- `.github/workflows/demo-command.yml:14`
- `.github/workflows/demo-command.yml:21`
- `.github/workflows/demo-command.yml:36`
- `.github/workflows/demo-command.yml:55`
- `.github/workflows/demo-command.yml:62`
- `.github/workflows/demo-command.yml:68`
- `.github/workflows/tag-command.yml:12`
- `.github/workflows/tag-command.yml:15`
- `.github/workflows/tag-command.yml:40`
- `.github/workflows/tag-command.yml:52`

### missing-permissions (severity: medium)

None of the four workflow files (chatops.yml, ci.yml, demo-command.yml, tag-command.yml) define a top-level `permissions:` key, and none of the individual jobs define job-level `permissions:` keys. Without explicit permissions, workflows run with the default (potentially broad) GITHUB_TOKEN permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/chatops.yml:1`
- `.github/workflows/ci.yml:1`
- `.github/workflows/demo-command.yml:1`
- `.github/workflows/tag-command.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across action.yml and the four workflow files:

1. script-injection (action.yml line 19): Moved `${{ github.action_path }}` from the run: command into env: as PWSH_SCRIPT_ACTION_PATH; run: now uses `$env:PWSH_SCRIPT_ACTION_PATH/action.ps1`.

2. script-injection (ci.yml lines 57, 65, 74, 77, 175): Moved all five step output/outcome expressions out of run: blocks into env: blocks (TEST_RESULT, TEST_OUTCOME, TEST_ERR_MSG, SET_ACTION_FAILED_OUTCOME), referencing them as $env:VAR_NAME in PowerShell.

3. unpinned-uses: Pinned all 13 action references to full SHA digests:
   - peter-evans/slash-command-dispatch@v1 → @ace7a198016ae74cd286677c7e7f7e266eb18bc4
   - actions/checkout@v2 → @0717577d45739eb3c851188b29f50ed6c0b2194e (×3)
   - peter-evans/create-or-update-comment@v1 → @a35cf36e5301d70b76f316e867e7788a55a31dae (×7)
   - Amadevus/pwsh-script@v1 → @aeda15d24afc567c90bf3f353bac8cb3016578a6 (×3)

4. missing-permissions: Added `permissions: contents: read` top-level block to chatops.yml, ci.yml, demo-command.yml, and tag-command.yml.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted PowerShell path expansion in action.yml line 20. Changed `$env:PWSH_SCRIPT_ACTION_PATH/action.ps1` to `& "$env:PWSH_SCRIPT_ACTION_PATH/action.ps1"`. Using PowerShell's call operator `&` with double-quoted path ensures the env var value (sourced from `github.action_path`) is treated as a single string even if it contains spaces or special characters, preventing unexpected command execution.

### Iteration 3

**Fixes applied:** invalid-yaml

**Notes:**

Fixed YAML parsing error at line 20 in action.yml. The `run:` value `& "$env:PWSH_SCRIPT_ACTION_PATH/action.ps1"` was causing a parse failure because `&` is a YAML anchor character and the quoted string following it confused the parser. Converted the single-line `run:` to a block scalar (`run: |`) so the PowerShell call operator and quoted path are treated as a literal string.

