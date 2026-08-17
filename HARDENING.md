<!-- markdownlint-disable -->

# Hardening Report: actions--setup-node/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-node/v7.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or branch names instead of pinned full-length SHA commits. This exposes the workflow to supply-chain attacks if the referenced tag or branch is updated with malicious code. Affected references include: `actions/reusable-workflows/.github/workflows/basic-validation.yml@main`, `actions/reusable-workflows/.github/workflows/check-dist.yml@main`, `actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main`, `actions/checkout@v6` (multiple files), `pnpm/action-setup@v4`, `actions/reusable-workflows/.github/workflows/licensed.yml@main`, `actions/publish-immutable-action@v0.0.4`, `actions/publish-action@v0.4.0`, `actions/reusable-workflows/.github/workflows/update-config-files.yml@main`.

Locations:

- `.github/workflows/basic-validation.yml:14`
- `.github/workflows/check-dist.yml:13`
- `.github/workflows/codeql-analysis.yml:12`
- `.github/workflows/e2e-cache.yml:20`
- `.github/workflows/licensed.yml:12`
- `.github/workflows/proxy.yml:20`
- `.github/workflows/publish-immutable-actions.yml:13`
- `.github/workflows/release-new-action-version.yml:20`
- `.github/workflows/update-config-files.yml:12`
- `.github/workflows/versions.yml:19`

### missing-permissions (severity: medium)

Several workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be broad), violating the principle of least privilege. Affected files: basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-cache.yml, licensed.yml, proxy.yml, update-config-files.yml, versions.yml.

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/e2e-cache.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/proxy.yml:1`
- `.github/workflows/update-config-files.yml:1`
- `.github/workflows/versions.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ matrix.node-version }}` (a workflow-controlled matrix context value) into shell command strings. Although matrix values here are statically defined, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell parses it, allowing shell metacharacters to be injected. Violating sub-rule (a). Affected patterns include: `run: __tests__/verify-node.sh "${{ matrix.node-version }}"` (e2e-cache.yml, multiple jobs), and multi-line scripts such as `canaryVersion="${{ matrix.node-version }}"`, `nightlyVersion="${{ matrix.node-version }}"`, `rcVersion="${{ matrix.node-version }}"`, and `nvm version-remote "${{ matrix.node-version }}"` (versions.yml).

Locations:

- `.github/workflows/e2e-cache.yml:28`
- `.github/workflows/e2e-cache.yml:55`
- `.github/workflows/e2e-cache.yml:82`
- `.github/workflows/e2e-cache.yml:113`
- `.github/workflows/e2e-cache.yml:218`
- `.github/workflows/e2e-cache.yml:247`
- `.github/workflows/versions.yml:22`
- `.github/workflows/versions.yml:50`
- `.github/workflows/versions.yml:72`
- `.github/workflows/versions.yml:88`
- `.github/workflows/versions.yml:107`
- `.github/workflows/versions.yml:126`
- `.github/workflows/versions.yml:145`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across 10 workflow files:

1. unpinned-uses: Pinned all external action references to full SHA commits using lookup_action_sha. Pinned actions/reusable-workflows@main, actions/checkout@v6, pnpm/action-setup@v4, actions/publish-immutable-action@v0.0.4, and actions/publish-action@v0.4.0. Also pinned ubuntu:latest and ubuntu/squid:latest container images in proxy.yml to their sha256 digests.

2. missing-permissions: Added `permissions: {}` top-level blocks to basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-cache.yml, licensed.yml, proxy.yml, update-config-files.yml, and versions.yml. The publish-immutable-actions.yml already had job-level permissions (contents: read, id-token: write, packages: write) and release-new-action-version.yml already had a top-level permissions: contents: write block.

3. script-injection: Moved all ${{ matrix.node-version }} expressions from run: shell strings into step-level env: blocks as NODE_VERSION, then referenced as "$NODE_VERSION" in the shell scripts. Fixed 6 locations in e2e-cache.yml and 7 locations in versions.yml (including the canaryVersion, nightlyVersion, rcVersion assignments and nvm version-remote call in lts-syntax job).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions inside command substitutions in .github/workflows/versions.yml: (1) v8-canary-syntax job line 83: `$(echo $canaryVersion | cut -d- -f1)` → `$(echo "$canaryVersion" | cut -d- -f1)`, (2) nightly-syntax job line 105: `$(echo $nightlyVersion | cut -d- -f1)` → `$(echo "$nightlyVersion" | cut -d- -f1)`, (3) rc-syntax job line 127: `$(echo $rcVersion | cut -d- -f1)` → `$(echo "$rcVersion" | cut -d- -f1)`. These variables hold the workflow-controllable matrix.node-version value (passed safely via env block), and quoting them inside the command substitution prevents shell metacharacter injection.

