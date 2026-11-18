<!-- markdownlint-disable -->

# Hardening Report: chuhlomin--render-template/v1.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **chuhlomin--render-template/v1.11** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The Docker action's runs.image points to a mutable tag, not a SHA digest: `image: "docker://ghcr.io/chuhlomin/render-template:v1.11"`. Whoever controls that tag can change what runs. Pin it to an @sha256: digest.

Locations:

- `action.yml:37`

### script-injection (severity: high)

Sub-rule (a): in the 'Download binary' step, expressions are placed straight into the run: script: `OS=$(echo "${{ runner.os }}" | tr '[:upper:]' '[:lower:]')` and `ARCH=$(echo "${{ runner.arch }}" | tr '[:upper:]' '[:lower:]')`. Use the $RUNNER_OS and $RUNNER_ARCH env vars instead.

Locations:

- `binary/action.yml:46`
- `binary/action.yml:48`

### script-injection (severity: high)

Sub-rule (a): in the 'Run' step, the whole run: script is an expression: `run: "${{ env.RENDER_TEMPLATE_BIN }}"`. The value comes from the env context, which can be set outside this action, and it is placed into the script before the shell runs. Use "$RENDER_TEMPLATE_BIN" instead.

Locations:

- `binary/action.yml:72`

### github-env-injection (severity: high)

The 'Download binary' step writes `echo "RENDER_TEMPLATE_BIN=$DEST/$BINARY" >> "$GITHUB_ENV"` without the printf '%s' ... | tr -d '\n\r' cleanup step. The value is built from the inherited env var $RUNNER_TEMP and from values taken from ${{ runner.os }} and ${{ runner.arch }}. None of these are literals computed inside the script.

Locations:

- `binary/action.yml:61`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Pinned the Docker image in action.yml to docker://ghcr.io/chuhlomin/render-template:v1.11@sha256:8a7257b4…, keeping the tag and the docker:// scheme. In binary/action.yml: ${{ runner.os }} and ${{ runner.arch }} are replaced with $RUNNER_OS and $RUNNER_ARCH. Newlines and carriage returns are now stripped from the binary path before it is written to GITHUB_ENV. The Run step now runs "$RENDER_TEMPLATE_BIN" (quoted) instead of the env-context expression. One extra change: the release download pointed at the action's own repo and ref, which is the mirror at runtime. It now uses ACTION_REPO='chuhlomin/render-template' and ACTION_REF='v1.11' (upstream's release), so the binary assets resolve.

