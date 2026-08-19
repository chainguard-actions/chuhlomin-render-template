<!-- markdownlint-disable -->

# Hardening Report: chuhlomin--render-template/v1.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **chuhlomin--render-template/v1.11** was hardened automatically. 7 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Download binary' step in binary/action.yml directly interpolates ${{ runner.os }} and ${{ runner.arch }} inside the run: shell script. Any ${{ ... }} expression inside a run: block is a script-injection risk because YAML template substitution happens before the shell ever sees the value. Offending lines: `OS=$(echo "${{ runner.os }}" | tr '[:upper:]' '[:lower:]')` and `ARCH=$(echo "${{ runner.arch }}" | tr '[:upper:]' '[:lower:]')`. These should be replaced with the safe env-var equivalents $RUNNER_OS and $RUNNER_ARCH.

Locations:

- `binary/action.yml:44`
- `binary/action.yml:46`

### script-injection (severity: high)

Sub-rule (a): The 'Run' step in binary/action.yml uses `run: "${{ env.RENDER_TEMPLATE_BIN }}"` — a ${{ ... }} expression is directly interpolated as the entire shell command string. This means the value of env.RENDER_TEMPLATE_BIN is substituted by the YAML template engine before the shell executes it, bypassing shell quoting protections. It should instead reference the env var as `run: "$RENDER_TEMPLATE_BIN"` (set via the env: block).

Locations:

- `binary/action.yml:65`

### unpinned-uses (severity: high)

The root action.yml uses a Docker image referenced by a mutable tag rather than a SHA digest: `image: "docker://ghcr.io/chuhlomin/render-template:v1.11"`. A tag can be silently overwritten to point to a different (potentially malicious) image. It should be pinned to a SHA256 digest, e.g. `docker://ghcr.io/chuhlomin/render-template@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:33`

### unpinned-uses (severity: high)

All uses: references in .github/workflows/main.yml use mutable tags instead of full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if any referenced action is compromised. Unpinned references: actions/checkout@v4 (line 14), actions/setup-go@v5 (line 17), golangci/golangci-lint-action@v6 (line 24), guyarb/golang-test-annotations@v0.8.0 (line 30), chuhlomin/actions/docker-build-push@main (line 34).

Locations:

- `.github/workflows/main.yml:14`
- `.github/workflows/main.yml:17`
- `.github/workflows/main.yml:24`
- `.github/workflows/main.yml:30`
- `.github/workflows/main.yml:34`

### unpinned-uses (severity: high)

All uses: references in .github/workflows/release.yml use mutable tags/versions instead of full 40-character commit SHAs. Unpinned references: actions/checkout@v6.0.2 (lines 13, 48, 63), docker/setup-buildx-action@v4.0.0 (line 19), docker/login-action@v4.1.0 (lines 22, 28), docker/build-push-action@v7.1.0 (line 34), actions/setup-go@v6.4.0 (line 51), goreleaser/goreleaser-action@v7.0.0 (line 55).

Locations:

- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:34`
- `.github/workflows/release.yml:48`
- `.github/workflows/release.yml:51`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:63`

### missing-permissions (severity: medium)

The workflow file .github/workflows/main.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow runs with the default (potentially broad) token permissions. A minimal permissions block (e.g. `permissions: contents: read`) should be added.

Locations:

- `.github/workflows/main.yml:1`

### missing-permissions (severity: medium)

The workflow file .github/workflows/release.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs (release, goreleaser, update-tag). Without explicit permissions, the workflow runs with the default (potentially broad) token permissions. Each job should declare the minimal permissions it requires.

Locations:

- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 7 findings:
1. binary/action.yml script-injection (line 44, 46): Replaced `${{ runner.os }}` and `${{ runner.arch }}` with built-in env vars `$RUNNER_OS` and `$RUNNER_ARCH`.
2. binary/action.yml script-injection (line 65): Replaced `"${{ env.RENDER_TEMPLATE_BIN }}"` with `"$RENDER_TEMPLATE_BIN"` (the var is already set in GITHUB_ENV by the prior step).
3. action.yml unpinned-uses (line 33): Pinned Docker image to `docker://ghcr.io/chuhlomin/render-template:v1.11@sha256:8a7257b4bc165216fbdce434f4d403f59ac3a639de748adaa90475d7ffd598ce`.
4. main.yml unpinned-uses (lines 14, 17, 24, 30, 34): Pinned all 5 actions to full commit SHAs (actions/checkout@11d5960a, actions/setup-go@40f1582b, golangci/golangci-lint-action@55c2c144, guyarb/golang-test-annotations@2941118d, chuhlomin/actions/docker-build-push@eae78b5d).
5. main.yml missing-permissions: Added top-level `permissions: contents: read`.
6. release.yml unpinned-uses (lines 13, 19, 22, 28, 34, 48, 51, 55, 63): Pinned all actions to full commit SHAs (actions/checkout@de0fac2e, docker/setup-buildx-action@4d04d5d9, docker/login-action@4907a6dd, docker/build-push-action@bcafcacb, actions/setup-go@4a360112, goreleaser/goreleaser-action@ec59f474).
7. release.yml missing-permissions: Added job-level permissions for each job: release (contents: read, packages: write), goreleaser (contents: write), update-tag (contents: write).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in `.github/workflows/main.yml` at the 'Get `result` output' step. Moved `${{ steps.render.outputs.result }}` from the `run:` shell string into an `env:` block as `RESULT`, and updated the shell command to reference `"$RESULT"` instead. This prevents the workflow-controllable output value from being interpolated directly into the shell command before execution.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed binary/action.yml line 55: Added sanitization of the RENDER_TEMPLATE_BIN value before writing to $GITHUB_ENV. The value is derived from $RUNNER_OS and $RUNNER_ARCH (workflow-controlled variables that could contain newlines). Now uses `RENDER_TEMPLATE_BIN_SAFE=$(printf '%s' "$DEST/$BINARY" | tr -d '\n\r')` and writes the sanitized value to $GITHUB_ENV, preventing newline injection attacks.

