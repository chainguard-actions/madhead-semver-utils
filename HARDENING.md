<!-- markdownlint-disable -->

# Hardening Report: madhead--semver-utils/v4.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **madhead--semver-utils/v4.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use mutable tag-based refs instead of pinned full SHA commits, making them vulnerable to supply-chain attacks if the referenced action tags are moved or compromised.

Failing references:
- default.yml: `actions/checkout@v4` (×2), `typesafegithub/github-actions-typing@v1`
- pr_main.yml: `actions/checkout@v4` (×2), `madhead/semver-utils@latest`
- release.yml: `actions/checkout@v4`, `stefanzweifel/git-auto-commit-action@v5`, `softprops/action-gh-release@v2`

Locations:

- `.github/workflows/default.yml:13`
- `.github/workflows/default.yml:107`
- `.github/workflows/default.yml:108`
- `.github/workflows/pr_main.yml:13`
- `.github/workflows/pr_main.yml:19`
- `.github/workflows/pr_main.yml:23`
- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:37`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and no job within them defines job-level permissions. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/default.yml:1`
- `.github/workflows/pr_main.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions inside shell command strings (sub-rule a). Although these particular contexts read from `steps.*.outputs.*`, `steps.*.outcome`, and `steps.*.conclusion` — which flow through YAML template substitution before the shell parses them — any expression interpolated directly in a `run:` block is a script-injection risk.

Failing lines in default.yml (representative sample):
- Line 24: `run: '[[ "failure" == "${{ steps.test-1.outcome }}" ]]'`
- Line 25: `run: '[[ "success" == "${{ steps.test-1.conclusion }}" ]]'`
- Line 38: `run: '[[ "" == "${{ steps.test-3.outputs.release }}" ]]'`
- Lines 39–42, 50–54, 57–69, 73, 77, 80–86 — similar patterns throughout

Failing lines in pr_main.yml:
- Line 28: `run: '[[ ">" == "${{ steps.comparison.outputs.comparison-result }}" ]]'`

Failing lines in release.yml:
- Line 27: `run: git tag --force v${{ steps.parsedVersion.outputs.major }}.${{ steps.parsedVersion.outputs.minor }}.${{ steps.parsedVersion.outputs.patch }}`
- Line 28: `run: git tag --force v${{ steps.parsedVersion.outputs.major }}.${{ steps.parsedVersion.outputs.minor }}`
- Line 29: `run: git tag --force v${{ steps.parsedVersion.outputs.major }}`

Locations:

- `.github/workflows/default.yml:24`
- `.github/workflows/default.yml:25`
- `.github/workflows/default.yml:32`
- `.github/workflows/default.yml:33`
- `.github/workflows/default.yml:38`
- `.github/workflows/default.yml:39`
- `.github/workflows/default.yml:40`
- `.github/workflows/default.yml:41`
- `.github/workflows/default.yml:42`
- `.github/workflows/default.yml:50`
- `.github/workflows/default.yml:51`
- `.github/workflows/default.yml:52`
- `.github/workflows/default.yml:53`
- `.github/workflows/default.yml:54`
- `.github/workflows/pr_main.yml:28`
- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:29`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across default.yml, pr_main.yml, and release.yml:

1. unpinned-uses: Pinned all 5 unique action references to full commit SHAs (actions/checkout@11d5960a3, typesafegithub/github-actions-typing@21db25f90, madhead/semver-utils@4cf918aff, stefanzweifel/git-auto-commit-action@b863ae193, softprops/action-gh-release@3bb12739c). Original tag preserved in inline comment.

2. missing-permissions: Added top-level `permissions: {}` to default.yml and pr_main.yml (no write access needed). Added `permissions: { contents: write }` to release.yml since it pushes git tags and creates GitHub releases.

3. script-injection: Moved all ${{ steps.*.outcome }}, ${{ steps.*.conclusion }}, and ${{ steps.*.outputs.* }} expressions out of run: shell strings into step-level env: blocks. Shell scripts now reference plain environment variables (e.g., $OUTCOME, $MAJOR, $COMPARISON_RESULT) instead of inline template expressions. The release.yml git tag commands now use MAJOR/MINOR/PATCH env vars instead of inline ${{ }} interpolation.

