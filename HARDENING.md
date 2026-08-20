<!-- markdownlint-disable -->

# Hardening Report: actions--jekyll-build-pages/v1.0.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--jekyll-build-pages/v1.0.13** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings. In docker-publish.yml the 'Generate Image Tags' step uses ${{ github.ref_name }}, ${{ env.REGISTRY }}, and ${{ env.IMAGE_NAME }} directly in shell code (lines 41-44), allowing an attacker-controlled branch/tag name to inject arbitrary shell commands. In release.yml the 'Version match' step uses ${{ steps.regex-match.outputs.group1 }} and ${{ env.TAG_NAME }} directly in shell conditionals (lines 37-38), and the 'Verify image published' step uses ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ env.TAG_NAME }} directly in a docker pull command (line 42). In test.yml the 'Verify output' step uses ${{matrix.test}} directly in a shell command (line 66).

Locations:

- `.github/workflows/docker-publish.yml:41`
- `.github/workflows/docker-publish.yml:44`
- `.github/workflows/release.yml:37`
- `.github/workflows/release.yml:42`
- `.github/workflows/test.yml:66`

### github-env-injection (severity: high)

In docker-publish.yml, the 'Generate Image Tags' run block builds a shell variable `tags` by directly interpolating ${{ github.ref_name }} (an attacker-controlled value via branch/tag name) and then writes it to $GITHUB_OUTPUT without any sanitization step (printf '%s' ... | tr -d '\n\r'). This allows newline injection into GITHUB_OUTPUT, potentially enabling environment variable injection for downstream steps. Offending line: `echo "tags=$tags" >> $GITHUB_OUTPUT` (line 46).

Locations:

- `.github/workflows/docker-publish.yml:46`

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use mutable tag-based or branch-based references instead of pinned full-length SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag or branch is moved or compromised.

docker-publish.yml: actions/checkout@v4, docker/login-action@v3, docker/metadata-action@v5, docker/build-push-action@v6
draft-release.yml: actions/checkout@v4, release-drafter/release-drafter@v6
record.yml: actions/checkout@v4, actions-ecosystem/action-regex-match@v2, docker/build-push-action@v6, actions/upload-artifact@v4
release.yml: actions/checkout@v4, actions-ecosystem/action-regex-match@v2, actions/publish-action@v0.3.0
shellcheck.yml: actions/checkout@v4, ludeeus/action-shellcheck@master (branch ref — especially dangerous)
test.yml: actions/checkout@v4, actions-ecosystem/action-regex-match@v2, docker/build-push-action@v6
action.yml: runs.image: 'docker://ghcr.io/actions/jekyll-build-pages:v1.0.13' (tag, not SHA digest)

Locations:

- `.github/workflows/docker-publish.yml:27`
- `.github/workflows/docker-publish.yml:32`
- `.github/workflows/docker-publish.yml:46`
- `.github/workflows/docker-publish.yml:52`
- `.github/workflows/draft-release.yml:13`
- `.github/workflows/draft-release.yml:14`
- `.github/workflows/record.yml:14`
- `.github/workflows/record.yml:24`
- `.github/workflows/record.yml:55`
- `.github/workflows/record.yml:62`
- `.github/workflows/release.yml:24`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:52`
- `.github/workflows/shellcheck.yml:14`
- `.github/workflows/shellcheck.yml:16`
- `.github/workflows/test.yml:21`
- `.github/workflows/test.yml:27`
- `.github/workflows/test.yml:51`
- `action.yml:31`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be read/write for all scopes), violating the principle of least privilege.

- docker-publish.yml: no permissions block at top level or job level
- record.yml: no permissions block at top level or job level
- shellcheck.yml: no permissions block at top level or job level
- test.yml: no permissions block at top level or job level

Locations:

- `.github/workflows/docker-publish.yml:1`
- `.github/workflows/record.yml:1`
- `.github/workflows/shellcheck.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four finding types across 7 files:

1. **unpinned-uses**: Pinned all action references to full commit SHAs in docker-publish.yml, draft-release.yml, record.yml, release.yml, shellcheck.yml, test.yml, and action.yml. Container image in action.yml pinned to sha256 digest while preserving docker:// scheme and tag.

2. **missing-permissions**: Added minimal permissions blocks to docker-publish.yml (contents: read, packages: write), record.yml (contents: read, packages: read), shellcheck.yml (contents: read), and test.yml (contents: read, packages: read).

3. **script-injection**: Moved all ${{ }} expressions out of run: shell strings into step env: blocks in docker-publish.yml (Generate Image Tags step), release.yml (Version match and Verify image published steps), and test.yml (Verify output step).

4. **github-env-injection**: In docker-publish.yml's Generate Image Tags step, sanitized the tags value (which incorporates attacker-controlled ref_name) with `printf '%s' "$tags" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

