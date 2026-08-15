<!-- markdownlint-disable -->

# Hardening Report: actions--setup-node/v3.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-node/v3.9.1** was hardened automatically. 0 finding(s) were identified and resolved across 1 iteration(s).

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across 10 workflow files:

1. script-injection: Moved all ${{ matrix.node-version }} and ${{ matrix.node-version-file }} expressions out of run: shell strings and into step-level env: blocks in versions.yml (8 locations) and e2e-cache.yml (4 locations). Shell scripts now reference plain env vars like $NODE_VERSION.

2. unpinned-uses: Pinned all mutable action references to full 40-char SHAs: actions/checkout@v3→a37ce91, actions/checkout@v4→11d5960, pnpm/action-setup@v2→eae0cfe, actions/publish-immutable-action@v0.0.4→4bc8754, actions/publish-action@v0.2.2→dca2315, actions/reusable-workflows@main→4735e71 (used in 5 reusable workflow callers).

3. missing-permissions: Added 'permissions: contents: read' top-level block to basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-cache.yml, licensed.yml, proxy.yml, update-config-files.yml, and versions.yml.

