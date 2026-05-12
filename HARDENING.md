# Hardening Report: chuhlomin--render-template/v1.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **chuhlomin--render-template/v1.12** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable tag instead of a SHA digest: `image: "docker://ghcr.io/chuhlomin/render-template:v1.12"`. A tag like `:v1.12` can be silently overwritten to point to a different (potentially malicious) image, enabling supply-chain attacks. The image reference should use a SHA256 digest, e.g. `docker://ghcr.io/chuhlomin/render-template@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `ghcr.io/chuhlomin/render-template:v1.12` with the immutable SHA256 digest `ghcr.io/chuhlomin/render-template@sha256:d3f85db65367419fd68511afeb78204c9394684a2b85353eed5d5a908774469c` in action.yml line 34. The original tag is preserved as a comment (# v1.12) outside the quoted string for readability.

