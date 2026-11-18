<!-- markdownlint-disable -->

# Hardening Report: chuhlomin--render-template/v1.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **chuhlomin--render-template/v1.12** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml is a Docker action whose runs.image uses a mutable tag instead of a SHA digest: `image: "docker://ghcr.io/chuhlomin/render-template:v1.12"`. Pin it to an @sha256 digest.

Locations:

- `action.yml:37`

### script-injection (severity: high)

Rule (a): the 'Download binary' step puts GitHub expressions straight into the run script: `OS=$(echo "${{ runner.os }}" | tr ...)` and `ARCH=$(echo "${{ runner.arch }}" | tr ...)`. Use the $RUNNER_OS and $RUNNER_ARCH env vars instead.

Locations:

- `binary/action.yml:46`
- `binary/action.yml:48`

### script-injection (severity: high)

Rule (a): the 'Run' step's whole script is an expression: `run: "${{ env.RENDER_TEMPLATE_BIN }}"`. Template substitution puts the env value into the shell script before bash parses it. Use `"$RENDER_TEMPLATE_BIN"` instead.

Locations:

- `binary/action.yml:89`

### github-env-injection (severity: high)

The 'Download binary' step writes `echo "RENDER_TEMPLATE_BIN=$DEST/$BINARY" >> "$GITHUB_ENV"` without running `printf '%s' ... | tr -d '\n\r'` first. DEST comes from the inherited $RUNNER_TEMP env var, and BINARY is built from runner.os and runner.arch expressions. Neither is a literal computed in the script, so the value must be sanitized before the write.

Locations:

- `binary/action.yml:78`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

I fixed all four findings in action.yml and binary/action.yml. Nothing has been run or tested yet.

- **unpinned-uses (action.yml):** the Docker image is now pinned to a digest, keeping the docker:// scheme and the tag: docker://ghcr.io/chuhlomin/render-template:v1.12@sha256:d3f85db65367419fd68511afeb78204c9394684a2b85353eed5d5a908774469c.
- **script-injection, 'Download binary' step:** the inline runner.os and runner.arch expressions are replaced with the $RUNNER_OS and $RUNNER_ARCH variables the runner already sets.
- **script-injection, 'Run' step:** the script is now '"$RENDER_TEMPLATE_BIN"', so the binary path comes from the environment variable instead of being pasted into the script text.
- **github-env-injection:** the binary path has newlines and carriage returns removed with printf | tr -d '\n\r' before it is written to $GITHUB_ENV.

One extra change you didn't list: the download step took its release repository and tag from the action's own repository and ref. On the mirror those point at the mirror, which has no releases, so the download would fail. I set ACTION_REPO to 'chuhlomin/render-template' and ACTION_REF to 'v1.12' (both quoted). Upstream's fallbacks (default repo when unset, resolving the latest 'v1.12.*' if 'v1.12' isn't a release) are unchanged.

