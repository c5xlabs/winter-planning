# Winter release automation decisions

## Release build baseline (decision:release-automation/release-build-baseline)

**Decider:** engineering

**Open.** Choose the minimum macOS versions, Linux libc/distribution baselines, and native or emulated CI execution
environments for the four agreed targets; this blocks finalizing `engineering:release-artifacts` and
`engineering:homebrew-distribution`. The execution plan must cover packaged-binary smoke tests and isolated Homebrew
installation, upgrade, and uninstall checks, not only cross-compilation.

## macOS release signing (decision:release-automation/macos-release-signing)

**Decider:** product owner and repository owner

**Open.** Decide whether the first macOS release requires Developer ID signing and notarization or only the signatures
needed to execute on the target platform; this blocks finalizing `engineering:release-artifacts`. Define the required
credentials and verification checks, including any difference between Homebrew installation and manually downloaded
archives.

## Distribution license (decision:release-automation/distribution-license)

**Decider:** product owner and repository owner

**Open.** Choose the code and binary distribution license for `c5xlabs/winter`, including the notices shipped in release
archives and the Homebrew formula's license metadata; this blocks `product:publish-platform-release` and
`engineering:release-artifacts`. The launcher repository currently contains only a README, so the agent must not infer
licensing from public repository visibility or copy the planning repository's license without approval.

## First release acceptance (decision:release-automation/first-release-acceptance)

**Decider:** product owner and engineering

**Open.** Under `decision:release-versioning`, decide whether this epic delivers the first public release and live tap
formula or a verified publication-ready workflow; this blocks delivery of `product:publish-platform-release` and
`engineering:homebrew-distribution`. Decide whether the first upgrade smoke test uses controlled prior-version fixtures
or requires two actual published versions, since no platform release exists yet.

## Homebrew publishing (decision:release-automation/homebrew-publishing)

**Decider:** engineering and repository owner

The platform's generated formula follows `decision:homebrew-channel` for tap ownership, publication order, scoped
automation credentials, and maintainer approval, so formula publication references only verified releases.

## Release automation (decision:release-automation/release-automation)

**Decider:** engineering and repository owner

Use `cargo-dist` for Rust CLI release builds, archives, checksums, GitHub Actions release workflows, GitHub Release
assets, and Homebrew formula generation rather than maintaining custom CLI release tooling. Repository-owned
verification remains responsible for executing packaged binaries on all four targets and proving that installations
exclude the development sample; tooling selection does not settle signing, versioning, or publishing credentials.

## Release targets (decision:release-automation/release-targets)

Initial prebuilt releases and Homebrew installation support macOS and Linux, each on x86_64 (AMD64) and ARM64 (aarch64),
covering the four developer-machine and server targets without adding Windows or 32-bit support. Intel macOS remains
included initially despite Homebrew's phase-out, with no Winter retirement date set; every target must run the packaged
executable's help and version smoke tests, since cross-compilation alone is not sufficient verification.
