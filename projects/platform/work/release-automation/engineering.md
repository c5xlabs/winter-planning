# Winter release automation engineering requirements

For `epic:release-automation`.

## Product release integration (engineering:release-integration)

Shared automation declares the inputs and hooks a product supplies for its version, source revision, artifacts,
verification, approval, and publication without imposing Rust CLI packaging on non-CLI artifacts. Serves
`product:shared-release-path` under `decision:shared-release-capability`.

Acceptance:

- The integration contract separates shared lifecycle checks from product-specific build, package, and publish steps.
- Documented adoption paths identify how Platform, Workspace, and Fleet connect their release artifacts and checks to
  that contract without copying the shared release policy into each product.
- Controlled fixtures exercise CLI and non-CLI artifact integration and prove that a failed required check prevents
  publication without requiring live product deployments or publishing credentials.
- Product-specific release integrations can adopt the shared capability independently; ordinary feature CI does not
  depend on the release pipeline or its credentials.

## Verified release artifacts (engineering:release-artifacts)

The release workflow builds versioned prebuilt archives from a recorded source revision and publishes them with
checksums after the required checks pass. Artifact targets and naming follow
`decision:release-automation/release-targets`, with automation following
`decision:release-automation/release-automation`. Release policy follows `decision:shared-release-governance` and
`decision:release-policy-baseline`. Serves `product:publish-platform-release`.

Acceptance:

- A documented version/tag operation triggers or prepares the release; the tag, package metadata, executable version,
  and artifact identifiers agree, and an inconsistent version fails the release checks.
- Published artifacts have checksums, contain the platform executable and required distribution material, and exclude
  development caches, credentials, and test-only executables.
- A release smoke check extracts the archive and runs help and version on each supported execution target; compilation
  alone is not treated as proof that a packaged binary runs.
- Packaging explicitly disables the foundation's development-sample feature and excludes the sample executable;
  archive and Homebrew checks prove that runtime settings and PATH entries cannot enable sample dispatch.
- The release workflow records the source revision and check results and does not silently replace an already published
  version with different artifacts.
- Ordinary build/test checks do not need publishing credentials; publishing permissions are scoped to the release path.

## Homebrew distribution (engineering:homebrew-distribution)

The Homebrew formula installs the matching verified prebuilt artifact for each supported Homebrew target, with its
checksum and version recorded in the formula. The tap and formula are owned through
`decision:homebrew-channel`. Serves `product:homebrew-installation`.

Acceptance:

- A documented release step updates the tap formula to the published version and checksums; a formula never references
  artifacts that have not been published.
- Formula checks and installation smoke tests verify help, version, upgrade, and uninstall using a controlled Homebrew
  environment, not the developer's live installation.
- Formula installation downloads prebuilt artifacts and requires no Rust compiler, Python runtime, or Winter product
  dependency.
- The code repository documents release prerequisites, the Homebrew update procedure, and the supported target matrix
  so publishing is repeatable rather than a collection of machine-local steps.
