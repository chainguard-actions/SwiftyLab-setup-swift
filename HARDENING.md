<!-- markdownlint-disable -->

# Hardening Report: SwiftyLab--setup-swift/v1.12.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **SwiftyLab--setup-swift/v1.12.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Every `uses:` reference in .github/workflows/main.yml is pinned to a mutable tag rather than a full 40-character commit SHA. This exposes the workflow to supply-chain attacks if any of these actions are compromised or their tags are moved. Unpinned references include: actions/checkout@v5, actions/setup-node@v4, actions/cache@v4.2.4, actions/github-script@v7, github/codeql-action/init@v3, github/codeql-action/analyze@v3, codecov/codecov-action@v5.5.1, addnab/docker-run-action@v3, actions/upload-artifact@v4, actions/configure-pages@v5, actions/upload-pages-artifact@v4, actions/deploy-pages@v4, crazy-max/ghaction-import-gpg@v6.3.0, TriPSs/conventional-changelog-action@v6, ncipollo/release-action@v1.

Locations:

- `.github/workflows/main.yml:35`
- `.github/workflows/main.yml:38`
- `.github/workflows/main.yml:41`
- `.github/workflows/main.yml:52`
- `.github/workflows/main.yml:72`
- `.github/workflows/main.yml:77`
- `.github/workflows/main.yml:80`
- `.github/workflows/main.yml:90`
- `.github/workflows/main.yml:93`
- `.github/workflows/main.yml:96`
- `.github/workflows/main.yml:104`

### permissions (severity: medium)

The workflow .github/workflows/main.yml has no top-level `permissions:` key, and the following jobs have no job-level `permissions:` block: `ci`, `unit-test`, `integration-test`, `dry-run`, `e2e-test`, `cd`. These jobs will inherit the default (potentially broad) repository permissions. Each job should declare the minimal permissions it requires.

Locations:

- `.github/workflows/main.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in .github/workflows/main.yml directly interpolate `${{ }}` expressions into shell commands, enabling script injection. (a) In the `dependabot` job, `github.event.pull_request.html_url` (attacker-controlled via PR) is interpolated directly into `gh pr review --approve "${{ github.event.pull_request.html_url }}"` and `gh pr merge --auto --squash "${{ github.event.pull_request.html_url }}"`; (b) In `integration-test`, `${{ steps.setup-swift.outputs.swift-version }}`, `${{ env.TOOLCHAINS || '""' }}`, `${{ matrix.sdk-url }}`, `${{ matrix.sdk-checksum }}`, and `${{ matrix.sdk }}` are all interpolated directly into shell commands (e.g. `xcrun --toolchain ${{ env.TOOLCHAINS || '""' }} swift --version | grep ${{ steps.setup-swift.outputs.swift-version }}`, `swift sdk install "${{ matrix.sdk-url }}" --checksum "${{ matrix.sdk-checksum }}"`, `swift build --swift-sdk ${{ matrix.sdk }}`); (c) In `dry-run`, `${{ steps.setup-swift.outputs.swift-version }}` is interpolated into a `grep` command; (d) In `e2e-test`, `${{ steps.setup-swift.outputs.swift-version }}` and `${{ env.TOOLCHAINS || '""' }}` are interpolated into shell commands. All these values should be passed via `env:` variables and then referenced as quoted shell variables.

Locations:

- `.github/workflows/main.yml:36`
- `.github/workflows/main.yml:37`
- `.github/workflows/main.yml:155`
- `.github/workflows/main.yml:158`
- `.github/workflows/main.yml:163`
- `.github/workflows/main.yml:167`
- `.github/workflows/main.yml:218`
- `.github/workflows/main.yml:248`
- `.github/workflows/main.yml:251`
- `.github/workflows/main.yml:257`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/main.yml:

1. **unpinned-uses**: Pinned all 15 action references to full commit SHAs with original tags as comments. SHAs resolved via lookup_action_sha for: actions/checkout@v5 (fbc6f39), actions/setup-node@v4 (49933ea), actions/cache@v4.2.4 (0400d5f), actions/github-script@v7 (f28e40c), github/codeql-action@v3 (08d09a5), codecov/codecov-action@v5.5.1 (5a10915), addnab/docker-run-action@v3 (4f65fab), actions/upload-artifact@v4 (ea165f8), actions/configure-pages@v5 (983d773), actions/upload-pages-artifact@v4 (7b1f4a7), actions/deploy-pages@v4 (d6db901), crazy-max/ghaction-import-gpg@v6.3.0 (e89d409), TriPSs/conventional-changelog-action@v6 (ee43def), ncipollo/release-action@v1 (339a818).

2. **permissions**: Added top-level `permissions: {}` and job-level permissions to all jobs missing them: ci/unit-test/integration-test/dry-run/e2e-test (contents: read), cd (contents: write), pages (id-token: write, pages: write, contents: read).

3. **script-injection**: Moved all attacker-controllable ${{ }} expressions from run: shell commands into env: blocks. Fixed: github.event.pull_request.html_url in dependabot job; steps.setup-swift.outputs.swift-version, env.TOOLCHAINS, matrix.sdk-url, matrix.sdk-checksum, matrix.sdk in integration-test; steps.setup-swift.outputs.swift-version in e2e-test. The dry-run job's addnab/docker-run-action with.run field retains ${{ steps.setup-swift.outputs.swift-version }} as it is a parameter to a docker action (not a shell step) and the value is from the action's own outputs (not attacker-controlled).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in hardened/action/.github/workflows/main.yml at the 'Verify Swift version' step in the 'dry-run' job. Moved `${{ steps.setup-swift.outputs.swift-version }}` from the `run:` shell string into an `env:` block as `SWIFT_VERSION`, and updated the shell command from `grep ${{ steps.setup-swift.outputs.swift-version }}` to `grep "$SWIFT_VERSION"` to prevent shell injection.

