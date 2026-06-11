<!-- markdownlint-disable -->

# Hardening Report: actions--jekyll-build-pages/v1.0.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions--jekyll-build-pages/v1.0.13** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml references a Docker image using a mutable version tag (`v1.0.13`) instead of an immutable SHA digest. This means the image could be replaced with a different (potentially malicious) version without changing the action reference. The failing reference is: `image: 'docker://ghcr.io/actions/jekyll-build-pages:v1.0.13'`. It should be pinned to a SHA digest, e.g. `image: 'docker://ghcr.io/actions/jekyll-build-pages@sha256:<64-hex-char-digest>'`.

Locations:

- `action.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag reference `docker://ghcr.io/actions/jekyll-build-pages:v1.0.13` with the immutable SHA256 digest `docker://ghcr.io/actions/jekyll-build-pages@sha256:6791ebfd912185ed59bfb5fb102664fa872496b79f87ff8b9cfba292a7345041` in action.yml line 30. The original tag is preserved as a comment (`# v1.0.13`) for readability.

