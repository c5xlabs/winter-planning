# Winter release automation requirements

For `epic:release-automation`.

## Reuse the shared release path (product:shared-release-path)

A maintainer can adopt a common release lifecycle for Platform, Workspace, and Fleet while retaining product-specific
artifacts and independent release versions under `decision:shared-release-capability`.

Acceptance:

- The shared path covers release preparation, required verification, approval, publication, and confirmation of the
  published result for all three projects.
- Product-specific integration supplies the artifacts and checks appropriate to CLI, service/container, or web outputs
  without inventing a separate versioning and approval policy for each packaging tool.
- Adopting the shared path does not require a simultaneous release of the other products.
- Existing Blizzard release procedures remain usable until their deliberate migration; feature development does not
  require release enablement to be delivered first.

## Publish an identifiable release (product:publish-platform-release)

A maintainer can publish a versioned platform executable that users can identify and obtain independently of the source
checkout. The initial supported targets are settled by `decision:release-automation/release-targets`.

Acceptance:

- A documented release procedure publishes prebuilt executables for the supported operating-system and CPU targets.
- A downloaded release reports the version under which it was published.
- A maintainer can identify the source revision and verification results associated with the release.
- Publishing the platform does not require releasing or installing Workspace or Fleet.
- Published archives, Homebrew installations, and installed help exclude the development-only sample executable and
  command registration, even when a sample executable or development settings are present on the user's machine.

Homebrew installation follows the suite's `product:homebrew-installation` requirement.
