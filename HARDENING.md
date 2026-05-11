# Hardening Report: chuhlomin--render-template/v1.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **chuhlomin--render-template/v1.10** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses a Docker image referenced by a mutable tag rather than an immutable SHA digest. In action.yml, `image: "docker://chuhlomin/render-template:v1.10"` uses the tag `v1.10`, which can be silently replaced with different (potentially malicious) content. It should be pinned to a specific SHA256 digest, e.g. `image: "docker://chuhlomin/render-template@sha256:<64-hex-char-digest>"`

Locations:

- `action.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable tag `chuhlomin/render-template:v1.10` to the immutable digest `chuhlomin/render-template@sha256:f34111a15ce5526a7733c1a481a442c6e0393a9d28ff804cc5e96b49bd6a3658 # v1.10`. The tag is preserved as a comment for readability.

