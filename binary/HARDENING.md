<!-- markdownlint-disable -->

# Hardening Report: chuhlomin--render-template--binary/v1.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **chuhlomin--render-template--binary/v1.11** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Download binary' step puts expressions straight into the run: script: `OS=$(echo "${{ runner.os }}" | tr '[:upper:]' '[:lower:]')` and `ARCH=$(echo "${{ runner.arch }}" | tr '[:upper:]' '[:lower:]')`. Use the $RUNNER_OS and $RUNNER_ARCH environment variables instead, double-quoted.

Locations:

- `action.yml:46`
- `action.yml:48`

### script-injection (severity: high)

Rule (a): the 'Run' step's script is the expression `run: "${{ env.RENDER_TEMPLATE_BIN }}"`. The env context is filled in before the shell runs, so whatever ends up in RENDER_TEMPLATE_BIN (set through GITHUB_ENV) runs as shell code. Use "$RENDER_TEMPLATE_BIN" instead.

Locations:

- `action.yml:72`

### github-env-injection (severity: high)

The 'Download binary' step runs `echo "RENDER_TEMPLATE_BIN=$DEST/$BINARY" >> "$GITHUB_ENV"` with no sanitization step. The value comes from the inherited $RUNNER_TEMP and from the ${{ runner.os }}/${{ runner.arch }} values, so it is not a literal computed inside the script. Remove newlines with printf '%s' ... | tr -d '\n\r' before writing it.

Locations:

- `action.yml:61`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all three findings in action.yml.
- Download step: it no longer puts ${{ runner.os }} or ${{ runner.arch }} straight into the script. It reads "$RUNNER_OS" and "$RUNNER_ARCH" instead, through printf '%s' | tr.
- Before writing RENDER_TEMPLATE_BIN to $GITHUB_ENV, the step now strips newline and carriage-return characters with printf '%s' | tr -d '\n\r'.
- Run step: replaced run: "${{ env.RENDER_TEMPLATE_BIN }}" with run: '"$RENDER_TEMPLATE_BIN"'.
- Also changed: this was not one of the findings. The step downloaded the binary from a release of the action's own repository, which on the mirror is the mirror repository. It now downloads from upstream, with ACTION_REPO set to 'chuhlomin/render-template' and ACTION_REF to 'v1.11' (upstream_version). Both values are quoted, and the ${ACTION_REPO:-...} fallback is kept.

