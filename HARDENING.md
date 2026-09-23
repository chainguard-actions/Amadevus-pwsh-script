<!-- markdownlint-disable -->

# Hardening Report: Amadevus--pwsh-script/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Amadevus--pwsh-script/v2.0.1** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is directly interpolated inside a `run:` shell command string. In action.yml, the step uses `run: ${{ github.action_path }}/action.ps1`, which injects the `github.action_path` context value directly into the shell command before the shell ever sees it. Per the check rules, ANY `${{ ... }}` expression inside a `run:` block is a script-injection finding, regardless of whether the specific context value is attacker-controlled. The safe alternative is to reference the pre-set `$GITHUB_ACTION_PATH` environment variable instead: `run: "$GITHUB_ACTION_PATH/action.ps1"`.

Locations:

- `action.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml line 19: replaced `run: ${{ github.action_path }}/action.ps1` with `run: & "$env:GITHUB_ACTION_PATH/action.ps1"`. Since the step uses `shell: pwsh`, the environment variable must be referenced using PowerShell syntax (`$env:GITHUB_ACTION_PATH`) and the script invoked with the call operator `&` with a quoted path to properly handle paths with spaces and avoid parse errors.

### Iteration 2

**Fixes applied:** invalid-yaml

**Notes:**

Fixed YAML parsing error on line 20 of action.yml. The `run: & "$env:GITHUB_ACTION_PATH/action.ps1"` was invalid YAML because the `&` character is a YAML anchor indicator and caused a parse failure. Converted the run value to a block scalar (`run: |`) so the PowerShell command is treated as literal text. The PowerShell command itself (`& "$env:GITHUB_ACTION_PATH/action.ps1"`) is preserved unchanged — it correctly uses the call operator `&` with a quoted path, which is the proper way to invoke a script with a path stored in an environment variable in PowerShell.

