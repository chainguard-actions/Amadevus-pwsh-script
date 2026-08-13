<!-- markdownlint-disable -->

# Hardening Report: Amadevus--pwsh-script/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Amadevus--pwsh-script/v1.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags instead of full 40-character SHA commit digests. This exposes the workflow to supply-chain attacks if the tag is moved. Failing references:
- .github/workflows/ci.yml: `actions/checkout@v2` (lines 17, 37, 52)
- .github/workflows/chatops.yml: `peter-evans/slash-command-dispatch@v1` (line 8)
- .github/workflows/tag-command.yml: `peter-evans/create-or-update-comment@v1` (lines 13, 39, 46), `actions/checkout@v2` (line 18)

Locations:

- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:37`
- `.github/workflows/ci.yml:52`
- `.github/workflows/chatops.yml:8`
- `.github/workflows/tag-command.yml:13`
- `.github/workflows/tag-command.yml:18`
- `.github/workflows/tag-command.yml:39`
- `.github/workflows/tag-command.yml:46`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no individual job defines its own `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions. All three workflow files are affected: ci.yml, chatops.yml, and tag-command.yml.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/chatops.yml:1`
- `.github/workflows/tag-command.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in ci.yml directly interpolate GitHub Actions expressions (`${{ ... }}`) inside shell command strings. Even though these reference `steps.*.outcome` and `steps.*.outputs.*` (which appear less attacker-controlled), any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk — the value is substituted by the template engine before the shell ever sees it, bypassing shell quoting. Offending lines:
- Line 62: `$outcome = '${{ steps.test-throwing-fails-action.outcome }}'`
- Line 67: `$errMsg = '${{ steps.test-throwing-fails-action.outputs.error }}'`
- Line 91: `$output = '${{ steps.Set-ActionOutput.outputs.test-output }}'`
- Line 143: `$result = '${{ steps.Set-ActionFailed.outcome }}'`
Fix: move the values into `env:` variables and reference them as `$env:VAR_NAME` in the PowerShell script.

Locations:

- `.github/workflows/ci.yml:62`
- `.github/workflows/ci.yml:67`
- `.github/workflows/ci.yml:91`
- `.github/workflows/ci.yml:143`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across ci.yml, chatops.yml, and tag-command.yml:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments: actions/checkout@v2 → ee0669bd1cc54295c223e0bb666b733df41de1c5, peter-evans/slash-command-dispatch@v1 → ace7a198016ae74cd286677c7e7f7e266eb18bc4, peter-evans/create-or-update-comment@v1 → a35cf36e5301d70b76f316e867e7788a55a31dae.

2. missing-permissions: Added top-level `permissions: {}` to all three workflow files. Added job-level minimal permissions where needed: chatops.yml dispatch job gets `issues: write`; tag-command.yml tag job gets `contents: write` and `issues: write`.

3. script-injection: Moved all four ${{ }} expressions out of run: shell scripts in ci.yml into env: blocks, referencing them as $env:OUTCOME, $env:ERR_MSG, $env:TEST_OUTPUT, and $env:RESULT in the PowerShell scripts.

