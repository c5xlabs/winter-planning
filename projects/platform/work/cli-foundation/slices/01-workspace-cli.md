---
name: workspace-cli
---

# Establish the Rust workspace and standalone CLI

Build the production interface and reproducible checkout setup for `product:discover-help`, `product:identify-version`,
`product:inspect-without-setup`, `product:unsupported-invocation`, `engineering:reproducible-setup`, and
`engineering:command-boundaries`, following [workspace boundaries](../spec/system.md) and
[CLI contracts](../spec/cli.md). This is additive work in the README-only `winter` repository, constrained by
`engineering:rust-product-clis`, `engineering:release-versions`, and `decision:release-delivery-separation`; no existing
CLI is replaced or migrated.

## Change

- Establish the four-package Cargo workspace, pinned toolchain and dependency lock, default production member, private
  implementation modules, and default-off development feature declarations.
- Implement the platform parsing/rendering facade and thin `winter` entry point: help, executable package version, and
  unsupported-invocation errors, with clap's standard help/version behavior.
- Keep sample dispatch absent; neither installed Winter components nor the Python Workspace CLI participate in parsing
  or execution.
- Provide the explicit bootstrap script, literal tool pins, repository-local tool installation, formatter/linter
  configuration, ignored build/tool output, and host prerequisites from
  [development setup](../spec/development.md#system). Document checkout build/run commands without installing `winter`.
- Add inline unit coverage for parsing, rendering, identity injection, and typed failures; keep process output/exit
  ownership at `main`.

## Acceptance

- A fresh checkout with documented host prerequisites bootstraps and builds with the pinned Rust toolchain and locked
  dependencies; a subsequent locked offline build leaves `Cargo.lock` unchanged and requires no Winter credentials.
- Cargo metadata identifies exactly the four packages and their permitted dependency direction; harness/sample packages
  are non-publishable and default builds select the production CLI.
- Bare invocation and explicit help produce identical stdout, empty stderr, and success; version matches `winter-cli`
  package metadata rather than the platform library, caller name, or working directory.
- Unknown inputs and unsupported product commands return explanatory stderr and failure; production help does not
  advertise sample dispatch. Unit tests exercise the same public facade used by the binary.
- A failed launcher reply write maps to exit `1`, no additional stdout, and a best-effort I/O diagnostic on stderr, as
  specified by the CLI reply contract; controlled unit checks cover that error mapping.
- Bootstrap uses the literal pinned versions, reuses matching local tools, verifies the actionlint archive checksum, and
  warms development caches without changing a shell profile or installing into the user's Cargo bin directory.
- Build/tool output remains untracked; running the checkout binary outside a workspace neither requires configuration
  nor changes the installed `winter` command or creates workspace files.

## Out of scope

- The shared process harness and executable regression suites belong to `process-test-harness`; aggregate gates and CI
  belong to `quality-gates`.
- Product discovery, Python compatibility, publication, and Homebrew remain outside this epic.
