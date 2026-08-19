<!-- markdownlint-disable -->

# Hardening Report: madhead--semver-utils/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **madhead--semver-utils/v4.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files use mutable tag/version refs instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved.

default.yml: `actions/checkout@v4` (×2), `typesafegithub/github-actions-typing@v1`
pr_main.yml: `actions/checkout@v4` (×2), `madhead/semver-utils@latest`
release.yml: `actions/checkout@v4`, `stefanzweifel/git-auto-commit-action@v5`, `softprops/action-gh-release@v2`

Locations:

- `.github/workflows/default.yml:12`
- `.github/workflows/default.yml:107`
- `.github/workflows/default.yml:108`
- `.github/workflows/pr_main.yml:13`
- `.github/workflows/pr_main.yml:19`
- `.github/workflows/pr_main.yml:27`
- `.github/workflows/release.yml:12`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:34`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no job within them defines job-level `permissions:` either. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/default.yml:1`
- `.github/workflows/pr_main.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks across all three workflow files directly interpolate `${{ ... }}` expressions inside shell command strings (sub-rule a). GitHub Actions performs template substitution before the shell parses the command, so any expression value containing shell metacharacters can break out of the intended command.

**default.yml**: Dozens of `run:` steps interpolate `${{ steps.test-N.outcome }}`, `${{ steps.test-N.conclusion }}`, and `${{ steps.test-N.outputs.* }}` directly into shell `[[ ... ]]` test commands. Example: `run: '[[ "failure" == "${{ steps.test-1.outcome }}" ]]'`

**pr_main.yml**: A `run:` step interpolates `${{ steps.comparison.outputs.comparison-result }}` directly into a shell `[[ ... ]]` command: `run: '[[ ">" == "${{ steps.comparison.outputs.comparison-result }}" ]]'`

**release.yml**: Three `run:` steps interpolate `${{ steps.parsedVersion.outputs.major }}`, `${{ steps.parsedVersion.outputs.minor }}`, and `${{ steps.parsedVersion.outputs.patch }}` directly into `git tag` shell commands. Example: `run: git tag --force v${{ steps.parsedVersion.outputs.major }}.${{ steps.parsedVersion.outputs.minor }}.${{ steps.parsedVersion.outputs.patch }}`

Locations:

- `.github/workflows/default.yml:22`
- `.github/workflows/default.yml:23`
- `.github/workflows/default.yml:31`
- `.github/workflows/default.yml:32`
- `.github/workflows/default.yml:38`
- `.github/workflows/default.yml:39`
- `.github/workflows/default.yml:40`
- `.github/workflows/default.yml:41`
- `.github/workflows/default.yml:42`
- `.github/workflows/pr_main.yml:33`
- `.github/workflows/release.yml:26`
- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:28`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three workflow files (.github/workflows/default.yml, pr_main.yml, release.yml):

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments preserved: actions/checkout@v4→11d5960a..., typesafegithub/github-actions-typing@v1→21db25f9..., madhead/semver-utils@latest→4cf918af..., stefanzweifel/git-auto-commit-action@v5→b863ae19..., softprops/action-gh-release@v2→3bb12739...

2. missing-permissions: Added top-level permissions blocks — `permissions: {}` for default.yml and pr_main.yml (no special permissions needed), and `permissions: { contents: write }` for release.yml (required for git push --tags and creating GitHub releases).

3. script-injection: Moved all ${{ }} expressions from run: shell strings into step env: blocks and referenced them as plain environment variables. In default.yml, consolidated multiple single-line test steps into multi-line run blocks with shared env: blocks. In pr_main.yml, moved comparison-result into COMPARISON_RESULT env var. In release.yml, moved major/minor/patch outputs into MAJOR/MINOR/PATCH env vars used in git tag commands.

