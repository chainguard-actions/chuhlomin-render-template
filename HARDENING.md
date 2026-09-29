<!-- markdownlint-disable -->

# Hardening Report: chuhlomin--render-template/v1.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **chuhlomin--render-template/v1.12** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a Docker image reference with a mutable tag (`docker://ghcr.io/chuhlomin/render-template:v1.12`) instead of an immutable SHA digest. This is vulnerable to supply-chain attacks if the tag is moved to a different image. It should be pinned to a SHA digest, e.g. `docker://ghcr.io/chuhlomin/render-template@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:34`

### script-injection (severity: high)

Sub-rule (a): binary/action.yml 'Download binary' step directly interpolates `${{ runner.os }}` and `${{ runner.arch }}` inside the `run:` shell script. Any `${{ ... }}` expression interpolated directly into a `run:` block undergoes YAML template substitution before the shell sees it, bypassing shell quoting and enabling script injection. Offending lines: `OS=$(echo "${{ runner.os }}" | tr '[:upper:]' '[:lower:]')` and `ARCH=$(echo "${{ runner.arch }}" | tr '[:upper:]' '[:lower:]')`. These should be passed via `env:` variables and referenced as `$RUNNER_OS` / `$RUNNER_ARCH` instead.

Locations:

- `binary/action.yml:46`
- `binary/action.yml:48`

### script-injection (severity: high)

Sub-rule (a): binary/action.yml 'Run' step uses `${{ env.RENDER_TEMPLATE_BIN }}` as the entire `run:` command string: `run: "${{ env.RENDER_TEMPLATE_BIN }}"`). This directly interpolates a GitHub Actions expression into the shell command before the shell parses it, enabling script injection if the env value contains shell metacharacters. It should be replaced with a shell variable reference, e.g. `run: "$RENDER_TEMPLATE_BIN"`.

Locations:

- `binary/action.yml:80`

### github-env-injection (severity: high)

binary/action.yml 'Download binary' step writes `echo "RENDER_TEMPLATE_BIN=$DEST/$BINARY" >> "$GITHUB_ENV"` without sanitizing the value first. `$DEST` is derived from `$RUNNER_TEMP` (an inherited process env var) and `$BINARY` is constructed from `$OS`/`$ARCH` which originate from `${{ runner.os }}`/`${{ runner.arch }}` interpolated directly in the script. A calling workflow could influence these values. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing newline injection into GITHUB_ENV which could set arbitrary environment variables for subsequent steps.

Locations:

- `binary/action.yml:69`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed four security findings in hardened/action:
1. action.yml: Pinned Docker image `ghcr.io/chuhlomin/render-template:v1.12` to immutable SHA digest `sha256:d3f85db65367419fd68511afeb78204c9394684a2b85353eed5d5a908774469c`, preserving the tag inline.
2. binary/action.yml (Download binary step): Moved `${{ runner.os }}` and `${{ runner.arch }}` out of the run script into the env block as `RUNNER_OS` and `RUNNER_ARCH`, then referenced them as `$RUNNER_OS`/`$RUNNER_ARCH` in the shell.
3. binary/action.yml (Download binary step): Added `safe=$(printf '%s' "$DEST/$BINARY" | tr -d '\n\r')` before writing to GITHUB_ENV to prevent newline injection.
4. binary/action.yml (Run step): Replaced `run: "${{ env.RENDER_TEMPLATE_BIN }}"` with `run: "$RENDER_TEMPLATE_BIN"` to use the shell environment variable directly instead of interpolating a GitHub Actions expression into the shell command.

