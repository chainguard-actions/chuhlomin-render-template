<!-- markdownlint-disable -->

# Hardening Report: chuhlomin--render-template/v1.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **chuhlomin--render-template/v1.9** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references and the Docker image in action.yml use mutable tags or branch names instead of pinned full 40-character SHA commit hashes or SHA digests, making the action vulnerable to supply-chain attacks.

action.yml: `image: 'docker://chuhlomin/render-template:v1.9'` — uses a mutable image tag instead of a SHA digest (e.g. `@sha256:<64-hex-char-digest>`).

.github/workflows/main.yml:
  - `uses: actions/checkout@v3` (tag ref)
  - `uses: chuhlomin/actions/docker-build-push@main` (branch ref)

.github/workflows/release.yml:
  - `uses: actions/checkout@v3` (tag ref, appears twice)
  - `uses: chuhlomin/actions/docker-build-push@main` (branch ref)

Locations:

- `action.yml:34`
- `.github/workflows/main.yml:14`
- `.github/workflows/main.yml:17`
- `.github/workflows/release.yml:14`
- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:33`

### script-injection (severity: high)

Rule (a) violation: A `run:` block in main.yml directly interpolates a `${{ }}` expression into a shell command string. The expression `${{ steps.render.outputs.result }}` is substituted into the shell command before the shell ever sees it, allowing an attacker who controls the rendered template output to inject arbitrary shell commands.

Offending line: `run: echo "${{ steps.render.outputs.result }}"`

Fix: pass the value through an environment variable and reference it as a quoted shell variable instead:
```yaml
env:
  RENDER_RESULT: ${{ steps.render.outputs.result }}
run: echo "$RENDER_RESULT"
```

Locations:

- `.github/workflows/main.yml:33`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` block, and no individual job within them defines job-level `permissions:` either. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions, which may be overly broad (e.g. `write` access to contents, packages, etc.). Every workflow should declare the minimal permissions required.

Affected files:
- .github/workflows/main.yml — no top-level or job-level permissions
- .github/workflows/release.yml — no top-level or job-level permissions (the `update-tag` job pushes tags and needs `contents: write`, but this should be scoped explicitly)

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

1. action.yml: Pinned Docker image 'chuhlomin/render-template:v1.9' to its SHA digest (sha256:1617f2bed4c2254f4d6835e91b0094bc493a24e7fe611d05aa03a4f303e970bd), preserving the 'docker://' scheme and tag inline.
2. main.yml: Pinned actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26 and chuhlomin/actions/docker-build-push@main → @eae78b5dcc38d93d9d5c5d40eb2c164740f3a764. Fixed script injection by moving ${{ steps.render.outputs.result }} into an env: block (RENDER_RESULT) and referencing it as $RENDER_RESULT in the shell. Added top-level 'permissions: {}'.
3. release.yml: Pinned both actions/checkout@v3 and chuhlomin/actions/docker-build-push@main to full SHAs. Added top-level 'permissions: {}', explicit 'permissions: {}' on the release job, and 'contents: write' on the update-tag job (which needs to push tags).

