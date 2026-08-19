<!-- markdownlint-disable -->

# Hardening Report: madhead--semver-utils/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **madhead--semver-utils/v4.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or version strings instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced tag is moved or the repository is compromised.

Failing references:
- .github/workflows/default.yml: `actions/checkout@v4`, `krzema12/github-actions-typing@v1`
- .github/workflows/pr_main.yml: `actions/checkout@v4`, `madhead/semver-utils@latest`
- .github/workflows/release.yml: `actions/checkout@v4`, `stefanzweifel/git-auto-commit-action@v4.16.0`, `softprops/action-gh-release@v1`

Locations:

- `.github/workflows/default.yml:13`
- `.github/workflows/default.yml:97`
- `.github/workflows/pr_main.yml:14`
- `.github/workflows/pr_main.yml:24`
- `.github/workflows/release.yml:14`
- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:33`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions inside shell command strings (sub-rule a). Even though many of these read from `steps.*.outputs.*` (which appear to be controlled values), any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, allowing special shell characters to be injected.

Examples of failing lines:
- default.yml: `run: '[[ "failure" == "${{ steps.test-1.outcome }}" ]]'`
- default.yml: `run: '[[ "1.2.3" == "${{ steps.test-5.outputs.release }}" ]]'`
- pr_main.yml: `run: '[[ ">" == "${{ steps.comparison.outputs.comparison-result }}" ]]'`
- release.yml: `run: git tag --force v${{ steps.parsedVersion.outputs.major }}.${{ steps.parsedVersion.outputs.minor }}.${{ steps.parsedVersion.outputs.patch }}`

All such expressions should be moved to `env:` blocks and the env vars double-quoted in the shell script.

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
- `.github/workflows/pr_main.yml:28`
- `.github/workflows/release.yml:26`
- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:29`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and none of the individual jobs define job-level `permissions:` keys. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege. Each workflow should declare the minimal permissions required (e.g., `permissions: read-all` at the top level and then grant specific write scopes per job as needed).

Locations:

- `.github/workflows/default.yml:1`
- `.github/workflows/pr_main.yml:1`
- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across default.yml, pr_main.yml, and release.yml:

1. unpinned-uses: Pinned all 5 external action references to full 40-char commit SHAs (actions/checkout, krzema12/github-actions-typing, madhead/semver-utils, stefanzweifel/git-auto-commit-action, softprops/action-gh-release) with tag comments for readability.

2. script-injection: Moved all ${{ }} expressions from run: shell strings into env: blocks, referencing them as plain $VAR_NAME environment variables. This covers ~30+ occurrences across the three files including test assertions in default.yml, the comparison check in pr_main.yml, and the git tag commands in release.yml.

3. missing-permissions: Added top-level `permissions: {}` to all three workflow files. For release.yml, added job-level `permissions: contents: write` to the release job since it needs to push tags and create GitHub releases.

