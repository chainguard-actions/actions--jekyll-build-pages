<!-- markdownlint-disable -->

# Hardening Report: actions--jekyll-build-pages/v1.0.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--jekyll-build-pages/v1.0.9** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions inside shell commands, violating sub-rule (a). In docker-publish.yml (lines 36–40), ${{ github.ref_name }}, ${{ env.REGISTRY }}, and ${{ env.IMAGE_NAME }} are interpolated directly in the shell script that builds Docker tags. In release.yml (lines 34–37), ${{ steps.regex-match.outputs.group1 }}, ${{ env.TAG_NAME }}, ${{ env.REGISTRY }}, and ${{ env.IMAGE_NAME }} are interpolated directly in run: blocks. In test.yml (line 57), ${{matrix.test}} is interpolated directly in a run: command: `./bin/compare_expected_output ./test_projects/${{matrix.test}}`.

Locations:

- `.github/workflows/docker-publish.yml:36`
- `.github/workflows/release.yml:34`
- `.github/workflows/release.yml:37`
- `.github/workflows/test.yml:57`

### github-env-injection (severity: high)

Several run: blocks write values derived from untrusted or workflow-controlled inputs to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). In docker-publish.yml (line 40), the shell variable $tags is constructed from ${{ github.ref_name }} (an attacker-controllable value via pull requests) and then written unsanitized: `echo "tags=$tags" >> $GITHUB_OUTPUT`. In record.yml (line 23), release.yml (line 29), and test.yml (line 23), the output of `grep` on action.yml is assigned to $image and written unsanitized: `echo "image=$image" >> $GITHUB_OUTPUT` — the grep result is workflow-controlled content that could contain newlines.

Locations:

- `.github/workflows/docker-publish.yml:40`
- `.github/workflows/record.yml:23`
- `.github/workflows/release.yml:29`
- `.github/workflows/test.yml:23`

### missing-permissions (severity: medium)

Four workflow files have no top-level permissions: key and no job-level permissions: keys on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be read/write for contents), violating the principle of least privilege. Affected files: docker-publish.yml, record.yml, shellcheck.yml, and test.yml.

Locations:

- `.github/workflows/docker-publish.yml:1`
- `.github/workflows/record.yml:1`
- `.github/workflows/shellcheck.yml:1`
- `.github/workflows/test.yml:1`

### unpinned-uses (severity: high)

Every uses: reference across all workflow files uses a mutable tag or branch ref instead of a pinned 40-character commit SHA, making the action vulnerable to supply-chain attacks if any upstream action is compromised or its tag is moved. Unpinned references include: actions/checkout@v3, docker/login-action@v2, docker/metadata-action@v4, docker/build-push-action@v4, actions-ecosystem/action-regex-match@v2, actions/upload-artifact@v3, release-drafter/release-drafter@v5, ludeeus/action-shellcheck@master (branch ref), and actions/publish-action@v0.2.2. Additionally, action.yml references a Docker image by mutable tag: `docker://ghcr.io/actions/jekyll-build-pages:v1.0.9` instead of a SHA digest.

Locations:

- `action.yml:30`
- `.github/workflows/docker-publish.yml:25`
- `.github/workflows/docker-publish.yml:30`
- `.github/workflows/docker-publish.yml:43`
- `.github/workflows/docker-publish.yml:49`
- `.github/workflows/draft-release.yml:13`
- `.github/workflows/draft-release.yml:14`
- `.github/workflows/record.yml:19`
- `.github/workflows/record.yml:24`
- `.github/workflows/record.yml:41`
- `.github/workflows/record.yml:44`
- `.github/workflows/record.yml:57`
- `.github/workflows/release.yml:23`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:55`
- `.github/workflows/shellcheck.yml:14`
- `.github/workflows/shellcheck.yml:17`
- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:41`
- `.github/workflows/test.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four finding categories across all workflow files and action.yml:

1. unpinned-uses: Pinned all 9 action references to full 40-char commit SHAs (actions/checkout, docker/login-action, docker/metadata-action, docker/build-push-action, actions-ecosystem/action-regex-match, actions/upload-artifact, release-drafter/release-drafter, ludeeus/action-shellcheck, actions/publish-action). Also pinned the Docker image in action.yml to its SHA digest (ghcr.io/actions/jekyll-build-pages:v1.0.9@sha256:24451ed8095993a68b5d343bd93837357f3f3c2b5e9aa42095f63b6228897bc6).

2. script-injection: Moved all ${{ }} expressions out of run: shell strings into step env: blocks in docker-publish.yml (github.ref_name, env.REGISTRY, env.IMAGE_NAME), release.yml (steps.regex-match.outputs.group1, env.TAG_NAME, env.REGISTRY, env.IMAGE_NAME), and test.yml (matrix.test).

3. github-env-injection: Added sanitization (printf '%s' "$var" | tr -d '\n\r') before writing to $GITHUB_OUTPUT in docker-publish.yml, record.yml, release.yml, and test.yml.

4. missing-permissions: Added top-level permissions blocks to docker-publish.yml (contents: read, packages: write), record.yml (contents: read, packages: read), shellcheck.yml (contents: read), and test.yml (contents: read, packages: read).

