<!-- markdownlint-disable -->

# Hardening Report: Amadevus--pwsh-script/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Amadevus--pwsh-script/v2.0.1** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in action.yml directly interpolates `${{ github.action_path }}` into the shell command string: `run: ${{ github.action_path }}/action.ps1`. Any GitHub Actions expression inside a `run:` block is a script-injection risk because YAML template substitution occurs before the shell ever sees the value.

Locations:

- `action.yml:21`

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in ci.yml directly interpolate GitHub Actions expressions into shell command strings. Examples include: `$result = '${{ steps.test-result-string.outputs.result }}'`, `$result = '${{ steps.test-result-object.outputs.result }}'`, `$outcome = '${{ steps.test-throwing-fails-action.outcome }}'`, `$errMsg = '${{ steps.test-throwing-fails-action.outputs.error }}'`, `$result = '${{ steps.Set-ActionFailed.outcome }}'`, `$result = '${{ steps.Invoke-ActionNoCommandsBlock.outputs.testout }}'`, and context values like `${{ matrix.os }}`, `${{ runner.temp }}`, `${{ strategy.fail-fast }}`, `${{ job.status }}`, `${{ github.token }}` interpolated directly in run: scripts.

Locations:

- `.github/workflows/ci.yml:20`
- `.github/workflows/ci.yml:55`
- `.github/workflows/ci.yml:65`
- `.github/workflows/ci.yml:76`
- `.github/workflows/ci.yml:83`
- `.github/workflows/ci.yml:113`
- `.github/workflows/ci.yml:121`
- `.github/workflows/ci.yml:163`
- `.github/workflows/ci.yml:171`

### script-injection (severity: high)

Sub-rule (a): In demo-command.yml, the step 'Execute user script' passes `${{ steps.get-script-text.outputs.result }}` directly as the `script:` input to the composite action (which executes it as a run: block). This allows the output of a prior step — which itself processes untrusted user-supplied comment body content — to be injected directly into a shell execution context without sanitization. Additionally, the 'Prettify result json' step uses `${{ steps.user-script.outputs.result }}` in an env: block but the env var name RESULT_JSON is fine; however the 'Comment with script result' step interpolates `${{ steps.pretty-result.outputs.result }}` and `${{ steps.user-script.outputs.error }}` directly into a body: string.

Locations:

- `.github/workflows/demo-command.yml:38`
- `.github/workflows/demo-command.yml:57`
- `.github/workflows/demo-command.yml:59`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable version tags instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: ci.yml — `actions/checkout@v2` (×3); chatops.yml — `peter-evans/slash-command-dispatch@v1`; demo-command.yml — `peter-evans/create-or-update-comment@v1` (×4), `Amadevus/pwsh-script@v1` (×2); tag-command.yml — `peter-evans/create-or-update-comment@v1` (×3), `actions/checkout@v2`.

Locations:

- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:33`
- `.github/workflows/ci.yml:41`
- `.github/workflows/chatops.yml:10`
- `.github/workflows/demo-command.yml:15`
- `.github/workflows/demo-command.yml:22`
- `.github/workflows/demo-command.yml:37`
- `.github/workflows/demo-command.yml:50`
- `.github/workflows/demo-command.yml:62`
- `.github/workflows/demo-command.yml:67`
- `.github/workflows/tag-command.yml:12`
- `.github/workflows/tag-command.yml:14`
- `.github/workflows/tag-command.yml:40`
- `.github/workflows/tag-command.yml:47`

### missing-permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` block, and no individual job within any of these files defines a `permissions:` block either. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/chatops.yml:1`
- `.github/workflows/demo-command.yml:1`
- `.github/workflows/tag-command.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across action.yml and 4 workflow files:

1. **script-injection (action.yml)**: Moved `${{ github.action_path }}` from the `run:` command string into an `env:` variable `ACTION_PATH`, using `$env:ACTION_PATH/action.ps1` in the run command.

2. **script-injection (ci.yml)**: Moved all step output/outcome references from `run:` blocks into `env:` blocks. For `uses:` steps that needed context values, added preceding `run:` steps to write values to `$GITHUB_ENV`. Fixed the Set-ActionOutput assert step (was guarded by `if: false` but still had injection).

3. **script-injection (demo-command.yml)**: Moved `steps.get-script-text.outputs.result` to env var `USER_SCRIPT` and used `Invoke-Expression $env:USER_SCRIPT`. Moved pretty-result and user-error outputs to env vars and referenced via `${{ env.* }}` in the body field.

4. **unpinned-uses**: Pinned all actions to full SHA commits:
   - `actions/checkout@v2` → `@ee0669bd1cc54295c223e0bb666b733df41de1c5 # v2`
   - `peter-evans/slash-command-dispatch@v1` → `@ace7a198016ae74cd286677c7e7f7e266eb18bc4 # v1`
   - `peter-evans/create-or-update-comment@v1` → `@a35cf36e5301d70b76f316e867e7788a55a31dae # v1`
   - `Amadevus/pwsh-script@v1` → `@aeda15d24afc567c90bf3f353bac8cb3016578a6 # v1`

5. **missing-permissions**: Added `permissions: {}` to all four workflow files (ci.yml, chatops.yml, demo-command.yml, tag-command.yml).

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed three github-env-injection findings in .github/workflows/ci.yml by adding PowerShell sanitization before writing workflow-controlled values to $GITHUB_ENV:
1. 'Set context expected values for context test' (line 113): Added `-replace '[\r\n]', ''` sanitization for all 5 values (GITHUB_TOKEN_VAL, JOB_STATUS, RUNNER_OS_VAL, STRATEGY_FAIL_FAST, MATRIX_OS) before writing to GITHUB_ENV.
2. 'Set Set-ActionFailed expected outcome' (line 183): Added `-replace '[\r\n]', ''` sanitization for RESULT before writing SET_ACTION_FAILED_RESULT to GITHUB_ENV.
3. 'Set Invoke-ActionNoCommandsBlock expected output' (line 208): Added `-replace '[\r\n]', ''` sanitization for TESTOUT before writing INVOKE_NO_COMMANDS_TESTOUT to GITHUB_ENV.
All sanitization uses PowerShell's -replace operator with a character class matching both \r and \n, which is the PowerShell equivalent of the bash `tr -d '\n\r'` approach.

