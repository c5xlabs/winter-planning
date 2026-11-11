# Suite engineering requirements

## Rust product CLIs (engineering:rust-product-clis)

All public-facing, released CLI tools that are part of the Winter product suite are implemented in Rust. This includes
the platform executable and launcher shim, Workspace CLI, and Fleet hub and runner CLIs; it does not constrain internal
development scripts or long-running services.

## Shared CLI runtime baseline (engineering:cli-runtime-baseline)

Released Rust product CLIs follow `decision:shared-cli-runtime-baseline` using the common target matrix and minimum
OS/libc baseline owned by shared release enablement, under `decision:platform/release-automation/release-targets` and
`decision:platform/release-automation/release-build-baseline`, rather than choosing incompatible baselines per product.
The minimum versions remain open in that shared decision; development CI host images are not release support promises.

Acceptance:

- Platform, Workspace, and Rust Fleet CLIs adopt the shared Linux/macOS × x86_64/aarch64 target baseline and the minimum
  versions once settled, without reopening the same OS/libc choice in each project.
- Each CLI's packaged executable is exercised on the agreed release baseline; a successful build on a newer CI host does
  not establish compatibility with the minimum system.
- Product-specific external tools and service-provider prerequisites are declared separately; they cannot silently
  change the CLI's shared runtime baseline or imply that every provider works on every supported CLI target.

## Release versions (engineering:release-versions)

Products adopting the shared release governance follow `decision:release-versioning`, with version numbers communicating
upgrade cost consistently rather than release cadence or epic completion.

Acceptance:

- From 1.0, patches contain backward-compatible fixes, minors add backward-compatible capabilities, and breaking changes
  require a major release.
- New products begin at `0.1.0`; existing products are not renumbered to adopt this policy.
- Before 1.0, patches remain backward-compatible, breaking changes require a new minor and explicit upgrade notes, and
  removing a released capability requires deprecation in an earlier release.
- Stable tags use `vX.Y.Z`; release-candidate tags use `vX.Y.Z-rc.N`.
- Package metadata and executable-reported versions match the full tagged version without its leading `v`, including the
  prerelease suffix where present.
- Release cadence and readiness for 1.0 are separate product decisions, not implied by this versioning policy.
