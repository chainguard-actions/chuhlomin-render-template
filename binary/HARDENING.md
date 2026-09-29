<!-- markdownlint-disable -->

# Hardening Report: chuhlomin--render-template--binary/v1.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **chuhlomin--render-template--binary/v1.12** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ runner.os }}` and `${{ runner.arch }}` are directly interpolated inside the `run:` shell script of the 'Download binary' step. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection risk — these should be accessed via the pre-set environment variables `$RUNNER_OS` and `$RUNNER_ARCH` instead. Offending lines:
  `OS=$(echo "${{ runner.os }}" | tr '[:upper:]' '[:lower:]')`
  `ARCH=$(echo "${{ runner.arch }}" | tr '[:upper:]' '[:lower:]')`

Locations:

- `action.yml:43`
- `action.yml:45`

### script-injection (severity: high)

Sub-rule (a): `${{ env.RENDER_TEMPLATE_BIN }}` is directly interpolated in the `run:` field of the 'Run' step: `run: "${{ env.RENDER_TEMPLATE_BIN }}"`  Any `${{ ... }}` expression directly inside a `run:` block is a script-injection finding. The binary path should be referenced as the shell environment variable `$RENDER_TEMPLATE_BIN` (set via `$GITHUB_ENV` in the prior step) rather than via template expression interpolation.

Locations:

- `action.yml:75`

### github-env-injection (severity: high)

The 'Download binary' step writes an inherited process env var (`$RUNNER_TEMP`) to `$GITHUB_ENV` without sanitization: `echo "RENDER_TEMPLATE_BIN=$DEST/$BINARY" >> "$GITHUB_ENV"`. `$DEST` is derived from `$RUNNER_TEMP`, which is an inherited runner environment variable (not set in this `run:` block) and is therefore workflow-controlled/untrusted for injection purposes. Writing it to `$GITHUB_ENV` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) violates check rule (e). A newline embedded in `$RUNNER_TEMP` could inject arbitrary entries into the GitHub environment file.

Locations:

- `action.yml:73`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three findings in hardened/action/action.yml:
1. Lines 43 & 45 (script-injection): Replaced `${{ runner.os }}` and `${{ runner.arch }}` with the pre-set environment variables `$RUNNER_OS` and `$RUNNER_ARCH`.
2. Line 73 (github-env-injection): Added sanitization before writing to $GITHUB_ENV: `SAFE_BIN=$(printf '%s' "$DEST/$BINARY" | tr -d '\n\r')` then writing `$SAFE_BIN` instead of the raw path.
3. Line 75 (script-injection): Replaced `"${{ env.RENDER_TEMPLATE_BIN }}"` with `"$RENDER_TEMPLATE_BIN"` to reference the shell environment variable set by the prior step rather than using a template expression inside the run block.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the 'Download binary' step of action.yml. The original code used `--jq "first(.[] | select(.tagName | startswith(\"${ACTION_REF}.\"))) | .tagName"` which interpolated the workflow-controllable `ACTION_REF` env var (from `github.action_ref`) unquoted inside a jq expression string, allowing shell metacharacter injection. The fix pipes `gh release list --json tagName` output to `jq -r --arg ref "${ACTION_REF}" 'first(.[] | select(.tagName | startswith($ref + "."))) | .tagName'`, passing `ACTION_REF` as a jq variable via `--arg` so it is treated as data, not code. The jq program itself is single-quoted with no shell interpolation.

