<!-- markdownlint-disable -->

# Hardening Report: chuhlomin--render-template/v1.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **chuhlomin--render-template/v1.13** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml is a Docker action whose runs.image uses a mutable tag instead of a SHA digest: image: "docker://ghcr.io/chuhlomin/render-template:v1.13". It should be pinned to a digest, e.g. ghcr.io/chuhlomin/render-template@sha256:<digest> # v1.13.

Locations:

- `action.yml:37`

### script-injection (severity: high)

Rule (a): binary/action.yml step 'Download binary' puts GitHub expressions straight into the run: script: `OS=$(echo "${{ runner.os }}" | tr ...)` and `ARCH=$(echo "${{ runner.arch }}" | tr ...)`. Use the $RUNNER_OS and $RUNNER_ARCH env vars instead.

Locations:

- `binary/action.yml:46`
- `binary/action.yml:48`

### script-injection (severity: high)

Rule (a): binary/action.yml step 'Run' puts an expression straight into the run: command: `run: "${{ env.RENDER_TEMPLATE_BIN }}"`. Use the shell variable instead, e.g. run: '"$RENDER_TEMPLATE_BIN"'.

Locations:

- `binary/action.yml:114`

### github-env-injection (severity: high)

binary/action.yml step 'Download binary' writes `echo "RENDER_TEMPLATE_BIN=$DEST/$BINARY" >> "$GITHUB_ENV"` with no sanitization. DEST is built from the inherited env var $RUNNER_TEMP, and BINARY is built from the runner.os/runner.arch expressions. Neither is passed through printf '%s' ... | tr -d '\n\r' before the write.

Locations:

- `binary/action.yml:103`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all four findings.

- action.yml: pinned the image by digest and kept the docker:// scheme and the tag: docker://ghcr.io/chuhlomin/render-template:v1.13@sha256:41e619a40f9db32747557eb8d3bc04f95c49d72dd7abb821b4ca880ffb3238b7.
- binary/action.yml, 'Download binary' step: OS and ARCH now come from $RUNNER_OS and $RUNNER_ARCH instead of inline ${{ runner.os }} / ${{ runner.arch }} expressions. The binary path is passed through tr -d '\n\r' before it is written to $GITHUB_ENV.
- binary/action.yml, 'Run' step: now runs '"$RENDER_TEMPLATE_BIN"' instead of an inline expression.

One change goes beyond the findings: ACTION_REPO and ACTION_REF are now fixed values, chuhlomin/render-template and the upstream commit c7bbd9cfdcdfe6ad595c5185286862c6f22567b4. They used to come from github.action_repository and github.action_ref, which name the mirror, and the binary releases this step downloads are published upstream. The script's existing commit-SHA branch then looks for an upstream release tag pointing at that commit. I have not checked that one exists; if none does, the step stops with 'No release tag points to commit ...'. Nothing was run or tested.

