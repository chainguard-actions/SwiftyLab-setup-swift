<!-- markdownlint-disable -->

# Hardening Report: SwiftyLab--setup-swift/v1.13.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **SwiftyLab--setup-swift/v1.13.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in .github/workflows/main.yml directly interpolate ${{ }} expressions into shell commands (sub-rule a), allowing script injection.

1. dependabot job 'Approve and Auto-merge' step: `gh pr review --approve "${{ github.event.pull_request.html_url }}"` and `gh pr merge --auto --squash "${{ github.event.pull_request.html_url }}"` — attacker-controlled PR URL injected directly into shell.

2. integration-test job 'Verify Swift version in macos' step: `xcrun --toolchain ${{ env.TOOLCHAINS || '""' }} swift --version | grep ${{ steps.setup-swift.outputs.swift-version }} || exit 1`

3. integration-test job 'Verify Swift version' step: `swift --version | grep ${{ steps.setup-swift.outputs.swift-version }} || exit 1`

4. integration-test job 'Install SDK' step: `swift sdk install "${{ matrix.sdk-url }}" --checksum "${{ matrix.sdk-checksum }}"`

5. integration-test job 'Test Swift package' step: `swift build --swift-sdk ${{ matrix.sdk }}`

6. dry-run job 'Verify Swift version' step (inside addnab/docker-run-action run:): `swift --version | grep ${{ steps.setup-swift.outputs.swift-version }} || exit 1` and `${{ steps.gen-sdk-install.outputs.command }}` — the latter executes arbitrary step output directly as a shell command.

7. e2e-test job 'Verify Swift version in macos' step: `xcrun --toolchain ${{ env.TOOLCHAINS || '""' }} swift --version | grep ${{ steps.setup-swift.outputs.swift-version }} || exit 1`

8. e2e-test job 'Verify Swift version' step: `swift --version | grep ${{ steps.setup-swift.outputs.swift-version }} || exit 1`

9. e2e-test job 'Verify Swift SDKs' step: `echo "$SWIFT_SDK_LIST" | grep ${{ steps.setup-swift.outputs.swift-version }}-RELEASE_static-linux || exit 1` (and similar lines for wasm and android).

Locations:

- `.github/workflows/main.yml:38`
- `.github/workflows/main.yml:39`
- `.github/workflows/main.yml:152`
- `.github/workflows/main.yml:153`
- `.github/workflows/main.yml:156`
- `.github/workflows/main.yml:163`
- `.github/workflows/main.yml:166`
- `.github/workflows/main.yml:230`
- `.github/workflows/main.yml:231`
- `.github/workflows/main.yml:270`
- `.github/workflows/main.yml:271`
- `.github/workflows/main.yml:273`
- `.github/workflows/main.yml:274`
- `.github/workflows/main.yml:275`

### unpinned-uses (severity: high)

Every uses: reference in .github/workflows/main.yml uses a mutable tag or version string instead of a pinned 40-character SHA commit hash, making the workflow vulnerable to supply-chain attacks.

Unpinned references include:
- actions/checkout@v6 (7 occurrences)
- actions/setup-node@v6 (6 occurrences)
- actions/cache@v5.0.3 (6 occurrences)
- actions/github-script@v8 (8 occurrences)
- github/codeql-action/init@v4
- github/codeql-action/analyze@v4
- codecov/codecov-action@v5.5.2
- addnab/docker-run-action@v3
- actions/upload-artifact@v7
- actions/configure-pages@v5
- actions/upload-pages-artifact@v4
- actions/deploy-pages@v4
- crazy-max/ghaction-import-gpg@v6.3.0
- TriPSs/conventional-changelog-action@v6
- ncipollo/release-action@v1

None are pinned to a full 40-character hex SHA digest.

Locations:

- `.github/workflows/main.yml:44`
- `.github/workflows/main.yml:47`
- `.github/workflows/main.yml:50`
- `.github/workflows/main.yml:63`
- `.github/workflows/main.yml:79`
- `.github/workflows/main.yml:83`
- `.github/workflows/main.yml:86`
- `.github/workflows/main.yml:99`
- `.github/workflows/main.yml:102`
- `.github/workflows/main.yml:105`
- `.github/workflows/main.yml:116`
- `.github/workflows/main.yml:148`
- `.github/workflows/main.yml:151`
- `.github/workflows/main.yml:154`
- `.github/workflows/main.yml:176`
- `.github/workflows/main.yml:213`
- `.github/workflows/main.yml:228`
- `.github/workflows/main.yml:231`
- `.github/workflows/main.yml:234`
- `.github/workflows/main.yml:244`
- `.github/workflows/main.yml:251`
- `.github/workflows/main.yml:260`
- `.github/workflows/main.yml:299`
- `.github/workflows/main.yml:340`
- `.github/workflows/main.yml:343`
- `.github/workflows/main.yml:346`
- `.github/workflows/main.yml:349`
- `.github/workflows/main.yml:358`
- `.github/workflows/main.yml:367`
- `.github/workflows/main.yml:372`
- `.github/workflows/main.yml:381`
- `.github/workflows/main.yml:395`
- `.github/workflows/main.yml:401`
- `.github/workflows/main.yml:404`
- `.github/workflows/main.yml:413`
- `.github/workflows/main.yml:430`
- `.github/workflows/main.yml:444`

### missing-permissions (severity: medium)

The workflow .github/workflows/main.yml has no top-level permissions: block, and the following jobs also lack job-level permissions: blocks: 'ci' (Check requirements), 'unit-test' (Run unit tests), 'integration-test' (Integrate Swift), 'dry-run' (Check action with dry run), 'e2e-test' (End-to-end test), and 'cd' (Create release). Without explicit permissions, these jobs inherit the default repository permissions, violating the principle of least privilege. Only 'dependabot', 'analyze', and 'pages' jobs have explicit permissions.

Locations:

- `.github/workflows/main.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/main.yml:

1. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks and referenced them as plain shell variables. Key fixes: dependabot PR_URL, integration-test SWIFT_VERSION/SDK_URL/SDK_CHECKSUM/MATRIX_SDK/TOOLCHAINS_VAL, dry-run SWIFT_VERSION/SDK_INSTALL_CMD (passed into docker container via options: -e), e2e-test SWIFT_VERSION/TOOLCHAINS_VAL. The dangerous ${{ steps.gen-sdk-install.outputs.command }} execution was moved to an env var SDK_INSTALL_CMD and executed via eval "$SDK_INSTALL_CMD" which is safe since the value is already controlled by the workflow's own github-script step.

2. unpinned-uses: Pinned all 15 unique action references to full 40-character commit SHAs with original tags preserved in comments. Verified SHAs using lookup_action_sha for each action.

3. missing-permissions: Added top-level permissions: {} and job-level permissions blocks to all jobs lacking them: ci (contents: read), unit-test (contents: read), integration-test (contents: read), dry-run (contents: read), e2e-test (contents: read), cd (contents: write for tag/release creation). Jobs that already had permissions (dependabot, analyze, pages) were left unchanged.

