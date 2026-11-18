<!-- markdownlint-disable -->

# Hardening Report: chuhlomin--render-template--binary/v1.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **chuhlomin--render-template--binary/v1.12** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Download binary' step puts GitHub Actions expressions straight into the run: script: `OS=$(echo "${{ runner.os }}" | tr '[:upper:]' '[:lower:]')` and `ARCH=$(echo "${{ runner.arch }}" | tr '[:upper:]' '[:lower:]')`. Use the $RUNNER_OS and $RUNNER_ARCH env vars instead.

Locations:

- `action.yml:46`
- `action.yml:48`

### script-injection (severity: high)

Rule (a): the 'Run' step's run: is `run: "${{ env.RENDER_TEMPLATE_BIN }}"`. The env.* expression is pasted into the shell script as text. Use "$RENDER_TEMPLATE_BIN" instead, since the step can already read it from the process environment.

Locations:

- `action.yml:89`

### github-env-injection (severity: high)

The 'Download binary' step writes `echo "RENDER_TEMPLATE_BIN=$DEST/$BINARY" >> "$GITHUB_ENV"` and does not strip newlines first. DEST comes from the inherited env var $RUNNER_TEMP, and BINARY comes from runner.os and runner.arch. Neither is a literal computed inside the script, so the value should first go through printf '%s' ... | tr -d '\n\r'.

Locations:

- `action.yml:78`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

All changes are in action.yml. I haven't run the action to test them.

1. Script injection, 'Download binary' step: the script now reads the runner's OS and architecture from $RUNNER_OS and $RUNNER_ARCH instead of having the ${{ runner.os }} and ${{ runner.arch }} expressions pasted into it.
2. Script injection, 'Run' step: the step now runs the binary as "$RENDER_TEMPLATE_BIN" instead of pasting in ${{ env.RENDER_TEMPLATE_BIN }}.
3. GITHUB_ENV injection: newlines are stripped from the binary path (printf '%s' ... | tr -d '\n\r') before it is written to $GITHUB_ENV.
4. Release download (a fix the findings didn't list): the step was downloading the binary from the action's own repository and ref, which on this mirror are the mirror's, and the mirror has no matching release. ACTION_REPO is now 'chuhlomin/render-template' and ACTION_REF is 'v1.12', both quoted so YAML reads them as text. The fallback for an unset repository and the lookup of the latest matching release are unchanged.

