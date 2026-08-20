<!-- markdownlint-disable -->

# Hardening Report: actions--jekyll-build-pages/v1.0.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--jekyll-build-pages/v1.0.12** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings. In docker-publish.yml, the 'Generate Image Tags' step uses `${{ github.ref_name }}`, `${{ env.REGISTRY }}`, and `${{ env.IMAGE_NAME }}` directly in the shell script. In release.yml, the 'Version match' step uses `${{ steps.regex-match.outputs.group1 }}` and `${{ env.TAG_NAME }}` directly in the shell, and the 'Verify image published' step uses `${{ env.REGISTRY }}`, `${{ env.IMAGE_NAME }}`, and `${{ env.TAG_NAME }}` directly in a docker pull command. These expressions are substituted by the Actions runner before the shell ever sees them, allowing injection of shell metacharacters.

Locations:

- `.github/workflows/docker-publish.yml:37`
- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:32`

### github-env-injection (severity: high)

In docker-publish.yml, the 'Generate Image Tags' step builds a shell variable `tags` by directly interpolating `${{ github.ref_name }}` and `${{ env.IMAGE_NAME }}` (which itself derives from `${{ github.repository }}`), then writes it to $GITHUB_OUTPUT via `echo "tags=$tags" >> $GITHUB_OUTPUT` without any sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker-controlled branch/tag name containing newlines could inject arbitrary entries into GITHUB_OUTPUT.

Locations:

- `.github/workflows/docker-publish.yml:42`

### unpinned-uses (severity: high)

Multiple workflow files and action.yml reference actions and Docker images by mutable tags/versions rather than full 40-character commit SHAs or SHA digests. Findings per file:

• action.yml: `image: 'docker://ghcr.io/actions/jekyll-build-pages:v1.0.12'` — tag reference, not a SHA digest.

• .github/workflows/docker-publish.yml: `actions/checkout@v4`, `docker/login-action@v3`, `docker/metadata-action@v5`, `docker/build-push-action@v5`.

• .github/workflows/draft-release.yml: `actions/checkout@v4`, `release-drafter/release-drafter@v6`.

• .github/workflows/record.yml: `actions/checkout@v4`, `actions-ecosystem/action-regex-match@v2`, `docker/build-push-action@v5`, `actions/upload-artifact@v4`.

• .github/workflows/release.yml: `actions/checkout@v4`, `actions-ecosystem/action-regex-match@v2`, `actions/publish-action@v0.3.0`.

• .github/workflows/shellcheck.yml: `actions/checkout@v4`, `ludeeus/action-shellcheck@master` (branch reference — especially dangerous).

• .github/workflows/test.yml: `actions/checkout@v4`, `actions-ecosystem/action-regex-match@v2`, `docker/build-push-action@v5`.

Locations:

- `action.yml:28`
- `.github/workflows/docker-publish.yml:25`
- `.github/workflows/docker-publish.yml:30`
- `.github/workflows/docker-publish.yml:47`
- `.github/workflows/docker-publish.yml:54`
- `.github/workflows/draft-release.yml:13`
- `.github/workflows/draft-release.yml:14`
- `.github/workflows/record.yml:24`
- `.github/workflows/record.yml:26`
- `.github/workflows/record.yml:40`
- `.github/workflows/record.yml:49`
- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:25`
- `.github/workflows/release.yml:46`
- `.github/workflows/shellcheck.yml:12`
- `.github/workflows/shellcheck.yml:14`
- `.github/workflows/test.yml:25`
- `.github/workflows/test.yml:27`
- `.github/workflows/test.yml:36`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` keys, meaning they run with the default (potentially broad) token permissions. Affected files: docker-publish.yml, shellcheck.yml, test.yml, and record.yml. Without explicit permissions, the GITHUB_TOKEN may have write access to contents, packages, and other scopes depending on repository settings.

Locations:

- `.github/workflows/docker-publish.yml:1`
- `.github/workflows/shellcheck.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/record.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings across 7 files:

1. script-injection: Moved ${{ github.ref_name }}, ${{ env.REGISTRY }}, ${{ env.IMAGE_NAME }} in docker-publish.yml Generate Image Tags step into env: block; moved ${{ steps.regex-match.outputs.group1 }}, ${{ env.TAG_NAME }}, ${{ env.REGISTRY }}, ${{ env.IMAGE_NAME }} in release.yml Version match and Verify image published steps into env: blocks.

2. github-env-injection: In docker-publish.yml Generate Image Tags step, all values are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before being written to $GITHUB_OUTPUT.

3. unpinned-uses: Pinned all 9 action references to full 40-char commit SHAs across docker-publish.yml, draft-release.yml, record.yml, release.yml, shellcheck.yml, and test.yml. Also pinned the Docker image in action.yml to its sha256 digest while preserving the docker:// scheme and tag.

4. missing-permissions: Added top-level permissions blocks to docker-publish.yml (contents: read, packages: write for Docker push), shellcheck.yml (contents: read), test.yml (contents: read), and record.yml (contents: read).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/test.yml line 55: moved `${{ matrix.test }}` out of the `run:` shell command and into the step's `env:` block as `MATRIX_TEST`. The shell script now references it safely as `"$MATRIX_TEST"` instead of directly interpolating the GitHub expression.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in all three workflow files (.github/workflows/test.yml, .github/workflows/record.yml, .github/workflows/release.yml). In each file's 'Grep action.yaml content' step, the grep output stored in `image` is now sanitized via `safe=$(printf '%s' "$image" | tr -d '\n\r')` before being written to $GITHUB_OUTPUT as `echo "image=$safe" >> "$GITHUB_OUTPUT"`. This prevents newline injection attacks where an attacker could modify action.yml in a PR to embed newlines in the matched content, injecting arbitrary key=value pairs into $GITHUB_OUTPUT.

