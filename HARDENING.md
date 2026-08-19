<!-- markdownlint-disable -->

# Hardening Report: chuhlomin--render-template/v1.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **chuhlomin--render-template/v1.12** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings in binary/action.yml. Line 46: `OS=$(echo "${{ runner.os }}" | tr ...)` — ${{ runner.os }} is substituted by the YAML template engine before the shell sees it, allowing injection. Line 48: `ARCH=$(echo "${{ runner.arch }}" | tr ...)` — same issue with ${{ runner.arch }}. Line 89: `run: "${{ env.RENDER_TEMPLATE_BIN }}"` — ${{ env.RENDER_TEMPLATE_BIN }} is interpolated directly as the shell command to execute, which is a critical injection point since env.* flows through YAML template substitution. All three should use environment variables ($RUNNER_OS, $RUNNER_ARCH, and a pre-set env var) instead of ${{ ... }} expressions inside run: blocks.

Locations:

- `binary/action.yml:46`
- `binary/action.yml:48`
- `binary/action.yml:89`

### script-injection (severity: high)

Sub-rule (a): The expression ${{ steps.render.outputs.result }} is interpolated directly inside a run: shell command string. Line 62: `run: echo "${{ steps.render.outputs.result }}"` — steps.*.outputs.* flows through YAML template substitution before the shell processes it, so any newlines or shell metacharacters in the output value could be interpreted by the shell. The value should be passed via an environment variable and then referenced as a quoted shell variable.

Locations:

- `.github/workflows/main.yml:62`

### unpinned-uses (severity: high)

Multiple uses: references in main.yml use mutable tags or branch names instead of pinned 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the referenced tag or branch is moved: `actions/checkout@v4` (line 16), `actions/setup-go@v5` (line 19), `golangci/golangci-lint-action@v6` (line 30), `guyarb/golang-test-annotations@v0.8.0` (line 39), `chuhlomin/actions/docker-build-push@main` (line 44, uses a branch name).

Locations:

- `.github/workflows/main.yml:16`
- `.github/workflows/main.yml:19`
- `.github/workflows/main.yml:30`
- `.github/workflows/main.yml:39`
- `.github/workflows/main.yml:44`

### unpinned-uses (severity: high)

Multiple uses: references in release.yml use mutable version tags instead of pinned 40-character commit SHAs: `actions/checkout@v6.0.2` (lines 16, 50, 70), `docker/setup-buildx-action@v4.0.0` (line 23), `docker/login-action@v4.1.0` (lines 26, 31), `docker/build-push-action@v7.1.0` (line 36), `actions/setup-go@v6.4.0` (line 55), `goreleaser/goreleaser-action@v7.0.0` (line 59). All should be pinned to full SHA digests.

Locations:

- `.github/workflows/release.yml:16`
- `.github/workflows/release.yml:23`
- `.github/workflows/release.yml:26`
- `.github/workflows/release.yml:31`
- `.github/workflows/release.yml:36`
- `.github/workflows/release.yml:50`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:59`
- `.github/workflows/release.yml:70`

### unpinned-uses (severity: high)

The docker action in action.yml references a mutable image tag instead of a SHA digest: `image: "docker://ghcr.io/chuhlomin/render-template:v1.12"`. The tag v1.12 can be overwritten at any time, allowing a supply-chain attack. The image reference should use a SHA digest, e.g. `docker://ghcr.io/chuhlomin/render-template@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:37`

### missing-permissions (severity: medium)

The workflow file main.yml has no top-level `permissions:` key and none of its jobs define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be write-all), granting the GITHUB_TOKEN broader access than necessary. A minimal permissions block (e.g. `permissions: contents: read`) should be added.

Locations:

- `.github/workflows/main.yml:1`

### missing-permissions (severity: medium)

The workflow file release.yml has no top-level `permissions:` key and none of its jobs (release, goreleaser, update-tag) define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions. Each job should declare only the permissions it needs (e.g. `contents: write` for the release job, `packages: write` for GHCR pushes).

Locations:

- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 7 findings across 4 files:

1. binary/action.yml (script-injection lines 46, 48, 89): Moved ${{ runner.os }} and ${{ runner.arch }} into env vars RUNNER_OS_INPUT/RUNNER_ARCH_INPUT; moved ${{ env.RENDER_TEMPLATE_BIN }} into env var RENDER_TEMPLATE_BIN and referenced as $RENDER_TEMPLATE_BIN in run command.

2. .github/workflows/main.yml (script-injection line 62): Moved ${{ steps.render.outputs.result }} into env var RENDER_RESULT.

3. .github/workflows/main.yml (unpinned-uses): Pinned all 5 action references to full commit SHAs.

4. .github/workflows/release.yml (unpinned-uses): Pinned all 6 action references (9 occurrences) to full commit SHAs.

5. action.yml (unpinned-uses): Pinned docker://ghcr.io/chuhlomin/render-template:v1.12 to SHA digest, preserving docker:// scheme and tag.

6. .github/workflows/main.yml (missing-permissions): Added top-level permissions: contents: read.

7. .github/workflows/release.yml (missing-permissions): Added per-job permissions blocks: release job gets contents: read + packages: write; goreleaser job gets contents: write; update-tag job gets contents: write.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in hardened/action/binary/action.yml line 69. Added sanitization step `safe=$(printf '%s' "$DEST/$BINARY" | tr -d '\n\r')` before writing to $GITHUB_ENV, replacing the direct `echo "RENDER_TEMPLATE_BIN=$DEST/$BINARY" >> "$GITHUB_ENV"` with `echo "RENDER_TEMPLATE_BIN=$safe" >> "$GITHUB_ENV"`. This prevents newline injection via the BINARY variable, which is derived from RUNNER_OS_INPUT and RUNNER_ARCH_INPUT (set from ${{ runner.os }} and ${{ runner.arch }}).

