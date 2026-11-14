# Winter component install decisions

## Component manifest (decision:component-install/component-manifest)

**Decider:** engineering and repository owner

**Open.** Whether each published archive carries the `component.toml` that `decision:launcher-shim/component-discovery`
requires, or `winter install` writes it after verification, and where the handoff version comes from if the installer
writes it; this blocks `product:install-component-version`.

## Release source (decision:component-install/release-source)

**Decider:** engineering and repository owner

`winter install` fetches each component from its product repository's GitHub Releases, using the archive and `.sha256`
asset that `cargo-dist` publishes under `decision:release-automation/release-automation`, and accepts a download on its
SHA-256 checksum alone. The releases already exist there, so a suite-wide index would be a second record to publish and
keep in step, and signature checks wait on `decision:release-automation/macos-release-signing`.
