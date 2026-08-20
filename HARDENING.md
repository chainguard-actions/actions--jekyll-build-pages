<!-- markdownlint-disable -->

# Hardening Report: actions--jekyll-build-pages/v1.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--jekyll-build-pages/v1.0.10** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use mutable tag-based references instead of pinned SHA commits, making them vulnerable to supply-chain attacks.

- action.yml: `image: docker://ghcr.io/actions/jekyll-build-pages:v1.0.10` (mutable tag, not a SHA digest)
- docker-publish.yml: `actions/checkout@v4`, `docker/login-action@v3`, `docker/metadata-action@v5`, `docker/build-push-action@v5`
- draft-release.yml: `actions/checkout@v4`, `release-drafter/release-drafter@v5`
- record.yml: `actions/checkout@v4`, `actions-ecosystem/action-regex-match@v2`, `docker/build-push-action@v5`, `actions/upload-artifact@v3`
- release.yml: `actions/checkout@v4`, `actions-ecosystem/action-regex-match@v2`, `actions/publish-action@v0.3.0`
- shellcheck.yml: `actions/checkout@v4`, `ludeeus/action-shellcheck@master`
- test.yml: `actions/checkout@v4`, `actions-ecosystem/action-regex-match@v2`, `docker/build-push-action@v5`

Locations:

- `action.yml:30`
- `.github/workflows/docker-publish.yml:26`
- `.github/workflows/docker-publish.yml:31`
- `.github/workflows/docker-publish.yml:44`
- `.github/workflows/docker-publish.yml:50`
- `.github/workflows/draft-release.yml:13`
- `.github/workflows/draft-release.yml:14`
- `.github/workflows/record.yml:20`
- `.github/workflows/record.yml:23`
- `.github/workflows/record.yml:37`
- `.github/workflows/record.yml:52`
- `.github/workflows/release.yml:24`
- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:51`
- `.github/workflows/shellcheck.yml:14`
- `.github/workflows/shellcheck.yml:16`
- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:23`
- `.github/workflows/test.yml:37`

### script-injection (severity: high)

GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings, enabling script injection.

(a) docker-publish.yml — `Generate Image Tags` step: `${{ github.ref_name }}` is interpolated directly into the shell script (both in the `if` condition and in the `tags=` assignment). An attacker controlling the branch/tag name could inject arbitrary shell commands.
  Offending lines:
  - `if [[ "${{ github.ref_name }}" == "main" ]]; then`
  - `tags=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.ref_name }}`

(a) release.yml — `Version match` step: `${{ steps.regex-match.outputs.group1 }}` and `${{ env.TAG_NAME }}` are interpolated directly into the shell `if` condition.
  Offending lines:
  - `if [ "${{ steps.regex-match.outputs.group1 }}" != "${{ env.TAG_NAME }}" ]; then`

(a) release.yml — `Verify image published` step: `${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ env.TAG_NAME }}` is interpolated directly into a `docker pull` command.
  Offending line:
  - `run: docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ env.TAG_NAME }}`

Locations:

- `.github/workflows/docker-publish.yml:38`
- `.github/workflows/docker-publish.yml:39`
- `.github/workflows/docker-publish.yml:41`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:37`

### github-env-injection (severity: high)

Untrusted values are written to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

- docker-publish.yml (`Generate Image Tags` step): The variable `tags` is assembled from `${{ github.ref_name }}` (attacker-controlled branch/tag name) and then written directly to `$GITHUB_OUTPUT` via `echo "tags=$tags" >> $GITHUB_OUTPUT`. No newline sanitization is applied, allowing newline injection to poison the output file.

- test.yml, record.yml, release.yml (`Grep action.yaml content` step): The shell variable `$image` (output of `grep`) is written to `$GITHUB_OUTPUT` via `echo "image=$image" >> $GITHUB_OUTPUT` without sanitization. The grep result is derived from repository file content, which is workflow-controllable, and no newline stripping is applied before the write.

Locations:

- `.github/workflows/docker-publish.yml:42`
- `.github/workflows/test.yml:28`
- `.github/workflows/record.yml:28`
- `.github/workflows/release.yml:28`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, workflows inherit the default repository token permissions, which may be overly broad (e.g., write access to contents).

- docker-publish.yml: no permissions block at all
- test.yml: no permissions block at all
- record.yml: no permissions block at all
- shellcheck.yml: no permissions block at all

Locations:

- `.github/workflows/docker-publish.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/record.yml:1`
- `.github/workflows/shellcheck.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four finding categories across action.yml and all 6 workflow files:

1. unpinned-uses: Pinned all action references to full commit SHAs and the container image to its sha256 digest (ghcr.io/actions/jekyll-build-pages:v1.0.10@sha256:8bf874abe806a08e923f9739fc3cbaa2e567d422a333e97e706c580234ca2cdc). All 10 unique actions pinned.

2. script-injection: Moved ${{ github.ref_name }} into an env var (REF_NAME) in docker-publish.yml's Generate Image Tags step. Moved ${{ steps.regex-match.outputs.group1 }} into env var (REGEX_GROUP1) in release.yml's Version match step. Replaced ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ env.TAG_NAME }} in docker pull with shell env var expansion ${REGISTRY}/${IMAGE_NAME}:${TAG_NAME}.

3. github-env-injection: Added printf '%s' "$var" | tr -d '\n\r' sanitization before all GITHUB_OUTPUT writes in docker-publish.yml, test.yml, record.yml, and release.yml.

4. missing-permissions: Added top-level permissions blocks to docker-publish.yml (contents: read, packages: write), test.yml (contents: read), record.yml (contents: read), and shellcheck.yml (contents: read). draft-release.yml and release.yml already had permissions blocks.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/test.yml at the 'Verify output' step. Moved `${{matrix.test}}` out of the run: shell command into an env: block as MATRIX_TEST, and updated the shell command to reference `$MATRIX_TEST` as a plain environment variable. The path argument is also now quoted to handle any spaces safely.

