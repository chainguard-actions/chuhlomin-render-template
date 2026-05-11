# Hardening Report: chuhlomin--render-template/v1.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **chuhlomin--render-template/v1.11** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable tag (`docker://ghcr.io/chuhlomin/render-template:v1.11`) instead of an immutable SHA digest. This means the image could be replaced with a different (potentially malicious) version without changing the action reference, creating a supply-chain risk. It should be pinned to a SHA digest, e.g. `docker://ghcr.io/chuhlomin/render-template@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `docker://ghcr.io/chuhlomin/render-template:v1.11` with the immutable SHA256 digest `docker://ghcr.io/chuhlomin/render-template@sha256:8a7257b4bc165216fbdce434f4d403f59ac3a639de748adaa90475d7ffd598ce` in action.yml line 33. The original tag `v1.11` is preserved as a comment for readability.

