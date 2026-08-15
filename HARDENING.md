<!-- markdownlint-disable -->

# Hardening Report: actions--setup-node/v6.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-node/v6.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Every workflow file uses mutable tag or branch refs instead of pinned 40-character SHA commits. This exposes the action to supply-chain attacks if any referenced action or reusable workflow is compromised or its tag is moved. Failing references include: `actions/reusable-workflows/...@main` (basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, update-config-files.yml), `actions/checkout@v6` (e2e-cache.yml, proxy.yml, versions.yml), `pnpm/action-setup@v4` (e2e-cache.yml), `actions/checkout@v6` and `actions/publish-immutable-action@v0.0.4` (publish-immutable-actions.yml), `actions/publish-action@v0.4.0` (release-new-action-version.yml).

Locations:

- `.github/workflows/basic-validation.yml:14`
- `.github/workflows/check-dist.yml:14`
- `.github/workflows/codeql-analysis.yml:13`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/update-config-files.yml:12`
- `.github/workflows/e2e-cache.yml:20`
- `.github/workflows/proxy.yml:22`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/release-new-action-version.yml:20`
- `.github/workflows/versions.yml:20`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no per-job `permissions:` keys. Without explicit permissions, workflows inherit the default (often broad) repository permissions, violating the principle of least privilege.

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

Sub-rule (a): Multiple `run:` blocks directly interpolate `${{ matrix.node-version }}` inside shell command strings. Although `matrix.*` values are typically developer-controlled, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted into the shell command string before the shell parses it, allowing shell metacharacters to be injected. Offending lines include: `run: __tests__/verify-node.sh "${{ matrix.node-version }}"` (versions.yml, e2e-cache.yml), `canaryVersion="${{ matrix.node-version }}"` (versions.yml), `nightlyVersion="${{ matrix.node-version }}"` (versions.yml), `rcVersion="${{ matrix.node-version }}"` (versions.yml), and `nvm version-remote "${{ matrix.node-version }}"` (versions.yml). The fix is to pass the value via an `env:` block and reference it as a quoted shell variable (e.g. `"$NODE_VERSION"`).

Locations:

- `.github/workflows/versions.yml:25`
- `.github/workflows/versions.yml:57`
- `.github/workflows/versions.yml:82`
- `.github/workflows/versions.yml:101`
- `.github/workflows/versions.yml:120`
- `.github/workflows/e2e-cache.yml:33`
- `.github/workflows/e2e-cache.yml:68`
- `.github/workflows/e2e-cache.yml:101`
- `.github/workflows/e2e-cache.yml:138`
- `.github/workflows/e2e-cache.yml:299`
- `.github/workflows/e2e-cache.yml:338`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across 10 workflow files:

1. **unpinned-uses**: Pinned all mutable refs to full 40-char SHAs:
   - actions/reusable-workflows@main → @4735e71081024a944852f4ab9d1495b6dd2de8f2 (5 files)
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 (versions.yml, e2e-cache.yml, proxy.yml, publish-immutable-actions.yml)
   - pnpm/action-setup@v4 → @b906affcce14559ad1aafd4ab0e942779e9f58b1 (e2e-cache.yml)
   - actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978 (publish-immutable-actions.yml)
   - actions/publish-action@v0.4.0 → @23f4c6f12633a2da8f44938b71fde9afec138fb4 (release-new-action-version.yml)

2. **missing-permissions**: Added `permissions: {}` top-level block to basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-cache.yml, licensed.yml, proxy.yml, update-config-files.yml, and versions.yml.

3. **script-injection**: Moved all `${{ matrix.node-version }}` expressions from `run:` shell strings into `env:` blocks as `NODE_VERSION: ${{ matrix.node-version }}` and updated shell scripts to reference `"$NODE_VERSION"` instead. Fixed all 11 locations across versions.yml (local-cache, lts-syntax, v8-canary-syntax, nightly-syntax, rc-syntax, manifest, check-latest, node-dist) and e2e-cache.yml (node-npm-depencies-caching, node-pnpm-depencies-caching, node-yarn1-depencies-caching, node-yarn3-depencies-caching, node-npm-packageManager-auto-cache, node-npm-devEngines-auto-cache).

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted variable expansions inside command substitutions in .github/workflows/versions.yml: (1) line 83 in v8-canary-syntax job: `echo $canaryVersion` → `echo "$canaryVersion"`; (2) line 105 in nightly-syntax job: `echo $nightlyVersion` → `echo "$nightlyVersion"`; (3) line 127 in rc-syntax job: `echo $rcVersion` → `echo "$rcVersion"`. The ${{ matrix.node-version }} expressions were already correctly placed in the env block; only the inner shell quoting was missing.

