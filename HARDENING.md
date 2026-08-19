<!-- markdownlint-disable -->

# Hardening Report: madhead--semver-utils/v4.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **madhead--semver-utils/v4.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference external actions using mutable tag refs instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if a tag is moved.

Failing references:
- .github/workflows/default.yml: `actions/checkout@v4` (×2), `typesafegithub/github-actions-typing@v1`
- .github/workflows/pr_main.yml: `actions/checkout@v4` (×2), `madhead/semver-utils@latest`
- .github/workflows/release.yml: `actions/checkout@v4`, `stefanzweifel/git-auto-commit-action@v5`, `softprops/action-gh-release@v2`

Locations:

- `.github/workflows/default.yml:13`
- `.github/workflows/default.yml:88`
- `.github/workflows/default.yml:89`
- `.github/workflows/pr_main.yml:14`
- `.github/workflows/pr_main.yml:21`
- `.github/workflows/pr_main.yml:26`
- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:34`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no job within them defines job-level permissions. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/default.yml:1`
- `.github/workflows/pr_main.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions inside shell command strings (rule a). GitHub Actions performs template substitution before the shell parses the string, so any expression containing shell metacharacters can break out of the intended command.

Failing steps in .github/workflows/default.yml: dozens of `run:` lines interpolate `${{ steps.test-N.outcome }}`, `${{ steps.test-N.conclusion }}`, and `${{ steps.test-N.outputs.* }}` directly in shell strings, e.g.:
  `- run: '[[ "failure" == "${{ steps.test-1.outcome }}" ]]'` (line 20)

Failing steps in .github/workflows/pr_main.yml:
  `- run: '[[ ">" == "${{ steps.comparison.outputs.comparison-result }}" ]]'` (line 30)

Failing steps in .github/workflows/release.yml:
  `- run: git tag --force v${{ steps.parsedVersion.outputs.major }}.${{ steps.parsedVersion.outputs.minor }}.${{ steps.parsedVersion.outputs.patch }}` (line 28)
  `- run: git tag --force v${{ steps.parsedVersion.outputs.major }}.${{ steps.parsedVersion.outputs.minor }}` (line 29)
  `- run: git tag --force v${{ steps.parsedVersion.outputs.major }}` (line 30)

All these should use env: variables and reference them as quoted shell variables instead.

Locations:

- `.github/workflows/default.yml:20`
- `.github/workflows/default.yml:21`
- `.github/workflows/default.yml:27`
- `.github/workflows/default.yml:28`
- `.github/workflows/pr_main.yml:30`
- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:29`
- `.github/workflows/release.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three workflow files:

1. unpinned-uses: Pinned all 9 action references to full 40-char SHAs with tag comments preserved: actions/checkout@v4→11d5960a..., typesafegithub/github-actions-typing@v1→21db25f9..., madhead/semver-utils@latest→4cf918af..., stefanzweifel/git-auto-commit-action@v5→b863ae19..., softprops/action-gh-release@v2→3bb12739...

2. missing-permissions: Added `permissions: {}` to default.yml and pr_main.yml (no token access needed). Added `permissions: contents: write` to release.yml (needs to push tags and create GitHub releases).

3. script-injection: Moved all ${{ }} expressions from run: shell strings into env: blocks and referenced them as plain shell variables ($VAR_NAME). This covers all test assertions in default.yml, the comparison check in pr_main.yml, and the git tag commands in release.yml. The softprops/action-gh-release with: block uses ${{ }} in action input values (not shell), which is safe and was left as-is.

