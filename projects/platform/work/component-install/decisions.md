# Winter component install decisions

## Release source (decision:component-install/release-source)

**Decider:** engineering and repository owner

**Open.** Where `winter install` finds published component releases and their verification data, such as each product
repository's GitHub Releases or a suite-wide release index, and how a published archive maps onto the version-cache
layout and its component manifest; this blocks `product:install-component-version` and `product:verified-install`.
Decide whether checksums alone are enough or signatures are required, in step with
`decision:release-automation/macos-release-signing`.
