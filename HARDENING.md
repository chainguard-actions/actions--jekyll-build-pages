<!-- markdownlint-disable -->

# Hardening Report: actions--jekyll-build-pages/v1.0.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--jekyll-build-pages/v1.0.11** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use mutable tag-based or branch-based refs instead of pinned 40-character SHA commits, making the action vulnerable to supply-chain attacks.

- action.yml: `image: 'docker://ghcr.io/actions/jekyll-build-pages:v1.0.11'` (tag, not SHA digest)
- docker-publish.yml: `actions/checkout@v4`, `docker/login-action@v3`, `docker/metadata-action@v5`, `docker/build-push-action@v5`
- draft-release.yml: `actions/checkout@v4`, `release-drafter/release-drafter@v6`
- record.yml: `actions/checkout@v4`, `actions-ecosystem/action-regex-match@v2`, `docker/build-push-action@v5`, `actions/upload-artifact@v4`
- release.yml: `actions/checkout@v4`, `actions-ecosystem/action-regex-match@v2`, `actions/publish-action@v0.3.0`
- shellcheck.yml: `actions/checkout@v4`, `ludeeus/action-shellcheck@master`
- test.yml: `actions/checkout@v4`, `actions-ecosystem/action-regex-match@v2`, `docker/build-push-action@v5`

Locations:

- `action.yml:30`
- `.github/workflows/docker-publish.yml:22`
- `.github/workflows/docker-publish.yml:28`
- `.github/workflows/docker-publish.yml:50`
- `.github/workflows/docker-publish.yml:57`
- `.github/workflows/draft-release.yml:11`
- `.github/workflows/draft-release.yml:12`
- `.github/workflows/record.yml:18`
- `.github/workflows/record.yml:22`
- `.github/workflows/record.yml:44`
- `.github/workflows/record.yml:57`
- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:24`
- `.github/workflows/release.yml:57`
- `.github/workflows/shellcheck.yml:14`
- `.github/workflows/shellcheck.yml:17`
- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:23`
- `.github/workflows/test.yml:44`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands, enabling script injection.

(a) docker-publish.yml 'Generate Image Tags' step: `if [[ "${{ github.ref_name }}" == "main" ]]; then` and `tags=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.ref_name }}` — github.ref_name is attacker-controlled (e.g. via branch/tag names) and is interpolated directly into the shell.

(a) release.yml 'Version match' step: `if [ "${{ steps.regex-match.outputs.group1 }}" != "${{ env.TAG_NAME }}" ]` and `echo "version mismatch...version ${{ steps.regex-match.outputs.group1 }} Tag version ${{ env.TAG_NAME }}"` — steps outputs and env context interpolated directly.

(a) release.yml 'Verify image published' step: `docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ env.TAG_NAME }}` — env context interpolated directly.

(a) test.yml 'Verify output' step: `./bin/compare_expected_output ./test_projects/${{matrix.test}}` — matrix context interpolated directly into shell command.

Locations:

- `.github/workflows/docker-publish.yml:36`
- `.github/workflows/docker-publish.yml:37`
- `.github/workflows/docker-publish.yml:39`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:35`
- `.github/workflows/release.yml:39`
- `.github/workflows/test.yml:55`

### github-env-injection (severity: high)

In docker-publish.yml, the 'Generate Image Tags' run: step builds the shell variable `tags` by directly interpolating `${{ github.ref_name }}` (an attacker-controlled value) via YAML template substitution, then writes it to $GITHUB_OUTPUT without sanitization: `echo "tags=$tags" >> $GITHUB_OUTPUT`. An attacker controlling the branch or tag name could inject newlines to poison GITHUB_OUTPUT with arbitrary key-value pairs. The required sanitization (`printf '%s' "$tags" | tr -d '\n\r'`) is absent.

Locations:

- `.github/workflows/docker-publish.yml:41`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning they run with the default (potentially broad) token permissions:

- docker-publish.yml: no permissions block at all
- record.yml: no permissions block at all
- shellcheck.yml: no permissions block at all
- test.yml: no permissions block at all

Locations:

- `.github/workflows/docker-publish.yml:1`
- `.github/workflows/record.yml:1`
- `.github/workflows/shellcheck.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four finding categories across 7 files:

1. unpinned-uses: Pinned all action refs to full 40-char SHAs (actions/checkout, docker/login-action, docker/metadata-action, docker/build-push-action, release-drafter/release-drafter, actions-ecosystem/action-regex-match, actions/upload-artifact, actions/publish-action, ludeeus/action-shellcheck) and pinned the container image in action.yml with its sha256 digest while preserving the docker:// scheme and tag.

2. script-injection: Moved all ${{ }} expressions out of run: blocks into step-level env: blocks in docker-publish.yml (github.ref_name, env.REGISTRY, env.IMAGE_NAME), release.yml (steps.regex-match.outputs.group1, env.TAG_NAME, env.REGISTRY, env.IMAGE_NAME), and test.yml (matrix.test → MATRIX_TEST).

3. github-env-injection: In docker-publish.yml Generate Image Tags step, the tags value (built from attacker-controlled REF_NAME) is now sanitized with `printf '%s' "$tags" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

4. missing-permissions: Added permissions blocks to docker-publish.yml (contents: read, packages: write), record.yml (contents: read), shellcheck.yml (contents: read), and test.yml (contents: read).

