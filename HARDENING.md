<!-- markdownlint-disable -->

# Hardening Report: madhead--semver-utils/v5.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **madhead--semver-utils/v5.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference external actions using mutable tag refs instead of pinned 40-character SHA commits, making them vulnerable to supply-chain attacks if a tag is moved.

Failing references:
- default.yml: `actions/checkout@v4` (×2), `typesafegithub/github-actions-typing@v1`
- pr_main.yml: `actions/checkout@v4` (×2), `madhead/semver-utils@latest`
- release.yml: `actions/checkout@v4`, `stefanzweifel/git-auto-commit-action@v5`, `softprops/action-gh-release@v3`

Locations:

- `.github/workflows/default.yml:13`
- `.github/workflows/default.yml:3746`
- `.github/workflows/default.yml:3780`
- `.github/workflows/pr_main.yml:14`
- `.github/workflows/pr_main.yml:21`
- `.github/workflows/pr_main.yml:25`
- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:35`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and no job within them defines job-level permissions either. This means workflows run with the default (potentially broad) token permissions granted by the repository settings.

Locations:

- `.github/workflows/default.yml:1`
- `.github/workflows/pr_main.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions (rule a), allowing shell metacharacter injection if the expression value contains special characters. The YAML template substitution happens before the shell ever sees the string.

In default.yml, dozens of run: steps interpolate `${{ steps.testN.outcome }}`, `${{ steps.testN.conclusion }}`, and `${{ steps.testN.outputs.* }}` directly into shell commands, e.g.:
  `run: '[[ "failure" == "${{ steps.test-1.outcome }}" ]]'`

In pr_main.yml (line 31):
  `run: '[[ ">" == "${{ steps.comparison.outputs.comparison-result }}" ]]'`

In release.yml (lines 30–32), step outputs are interpolated directly into git tag commands:
  `run: git tag --force v${{ steps.parsedVersion.outputs.major }}.${{ steps.parsedVersion.outputs.minor }}.${{ steps.parsedVersion.outputs.patch }}`
  `run: git tag --force v${{ steps.parsedVersion.outputs.major }}.${{ steps.parsedVersion.outputs.minor }}`
  `run: git tag --force v${{ steps.parsedVersion.outputs.major }}`

All these should be moved to `env:` variables and then referenced as double-quoted shell variables.

Locations:

- `.github/workflows/default.yml:22`
- `.github/workflows/default.yml:23`
- `.github/workflows/default.yml:30`
- `.github/workflows/default.yml:31`
- `.github/workflows/pr_main.yml:31`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:31`
- `.github/workflows/release.yml:32`

### github-env-injection (severity: high)

In pr_main.yml (lines 18 and 23) and release.yml (line 22), `node -p` is used to extract the version from `package.json` and the result is written directly to `$GITHUB_OUTPUT` without sanitization:
  `run: node -p "'version=' + require('./package.json').version" >> $GITHUB_OUTPUT`

The value written is derived from `package.json`, which is workflow-controlled content (e.g. from a PR branch). A malicious `package.json` version string containing newlines could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs. The sanitization step (`printf '%s' "$VALUE" | tr -d '\n\r'`) must be applied before writing to the special environment file.

Locations:

- `.github/workflows/pr_main.yml:18`
- `.github/workflows/pr_main.yml:23`
- `.github/workflows/release.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across default.yml, pr_main.yml, and release.yml:

1. unpinned-uses: Pinned all 5 distinct action references to full 40-char SHAs (actions/checkout→11d5960a, typesafegithub/github-actions-typing→21db25f9, madhead/semver-utils→4cf918af, stefanzweifel/git-auto-commit-action→b863ae19, softprops/action-gh-release→3d0d9888), preserving the original tag in a trailing comment.

2. missing-permissions: Added top-level `permissions: {}` to default.yml and pr_main.yml; added `permissions: contents: write` to release.yml (required for git push --tags and gh-release).

3. script-injection: Moved all ${{ steps.*.outcome }}, ${{ steps.*.conclusion }}, ${{ steps.*.outputs.* }}, and ${{ steps.comparison.outputs.comparison-result }} expressions out of run: shell strings into step-level env: blocks, referencing them as plain $VAR_NAME shell variables. Consolidated related single-line test steps into multi-line run blocks where appropriate.

4. github-env-injection: Replaced direct `node -p "'version=' + ..." >> $GITHUB_OUTPUT` with a two-step approach: capture the raw value, sanitize with `printf '%s' "$raw_version" | tr -d '\n\r'`, then write the safe value to $GITHUB_OUTPUT. Applied to base-version and new-version steps in pr_main.yml and the version step in release.yml.

