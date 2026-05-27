# Hardening Report: actions--jekyll-build-pages/v1.0.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions--jekyll-build-pages/v1.0.13** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The `runs.image:` field in action.yml references the Docker image `docker://ghcr.io/actions/jekyll-build-pages:v1.0.13` using a mutable version tag (`v1.0.13`) rather than an immutable SHA digest. This means the image could be silently replaced with a different (potentially malicious) version without any change to the action's source code, enabling a supply-chain attack. It should be pinned to a specific SHA digest, e.g. `image: ghcr.io/actions/jekyll-build-pages@sha256:<64-hex-char-digest>`

Locations:

- `action.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag reference `docker://ghcr.io/actions/jekyll-build-pages:v1.0.13` with the immutable SHA256 digest `docker://ghcr.io/actions/jekyll-build-pages@sha256:6791ebfd912185ed59bfb5fb102664fa872496b79f87ff8b9cfba292a7345041` in action.yml line 32. The original tag `v1.0.13` is preserved as a comment outside the YAML quotes for readability.

