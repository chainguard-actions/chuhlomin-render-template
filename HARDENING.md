<!-- markdownlint-disable -->

# Hardening Report: chuhlomin--render-template/v1.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **chuhlomin--render-template/v1.10** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or branch names instead of immutable 40-character SHA commits, making the workflow vulnerable to supply-chain attacks. Failing references in main.yml: `actions/checkout@v4`, `actions/setup-go@v5`, `golangci/golangci-lint-action@v4`, `guyarb/golang-test-annotations@v0.7.0`, `chuhlomin/actions/docker-build-push@main`. Failing references in release.yml: `actions/checkout@v4` (×2), `chuhlomin/actions/docker-build-push@main`. Additionally, action.yml uses a Docker image with a mutable tag instead of a SHA digest: `image: "docker://chuhlomin/render-template:v1.10"`.

Locations:

- `.github/workflows/main.yml:14`
- `.github/workflows/main.yml:17`
- `.github/workflows/main.yml:22`
- `.github/workflows/main.yml:28`
- `.github/workflows/main.yml:33`
- `.github/workflows/release.yml:14`
- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:37`
- `action.yml:34`

### permissions (severity: medium)

missing-permissions: Neither .github/workflows/main.yml nor .github/workflows/release.yml defines a top-level `permissions:` key, and no job within either file defines its own `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Rule (a) violation: A `run:` block in main.yml directly interpolates `${{ steps.render.outputs.result }}` into a shell command string. The `steps.*.outputs.*` context is a workflow-controllable value that flows through YAML template substitution before the shell sees it, enabling script injection if the rendered output contains shell metacharacters. Offending line: `run: echo "${{ steps.render.outputs.result }}"`

Locations:

- `.github/workflows/main.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings:
1. unpinned-uses: Pinned all action references in main.yml and release.yml to full 40-char SHAs (actions/checkout, actions/setup-go, golangci/golangci-lint-action, guyarb/golang-test-annotations, chuhlomin/actions/docker-build-push). Also pinned the Docker image in action.yml to its SHA digest while preserving the docker:// scheme and tag.
2. permissions: Added top-level `permissions: contents: read` to main.yml and `permissions: contents: write` to release.yml (write is required for the update-tag job that force-pushes the v1 tag).
3. script-injection: Moved `${{ steps.render.outputs.result }}` from the run: shell string into an env: block as RENDER_RESULT, then referenced it safely as `$RENDER_RESULT` in the shell command.

