# Hardening Report: chuhlomin--render-template/v1.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **chuhlomin--render-template/v1.9** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses a Docker image referenced by a mutable tag (`docker://chuhlomin/render-template:v1.9`) instead of an immutable SHA digest. This means the image could be replaced with a different (potentially malicious) version without changing the action.yml reference. It should be pinned to a specific SHA digest, e.g. `image: chuhlomin/render-template@sha256:<64-hex-char-digest> # v1.9`.

Locations:

- `action.yml:37`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `chuhlomin/render-template:v1.9` with the immutable SHA digest `chuhlomin/render-template@sha256:1617f2bed4c2254f4d6835e91b0094bc493a24e7fe611d05aa03a4f303e970bd` in action.yml line 37. The original tag is preserved as a comment for readability.

