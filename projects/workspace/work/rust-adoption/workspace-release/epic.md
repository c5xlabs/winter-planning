---
epic: rust-adoption/workspace-release
refinement: outline
status: planned
---

# Install the Workspace replacement

Part of `initiative:rust-adoption`; product requirements live in [requirements.md](./requirements.md).

## The problem

An engineer should not need a Rust compiler, a Python environment, or a Fleet installation to use the replacement.
Publishing an executable without its documentation, providers, and upgrade guidance would still leave that engineer
assembling the product by hand.

## The outcome

The engineer installs a prebuilt Workspace component through the suite's supported channel, uses it independently of
Fleet, and upgrades with an explicit account of compatibility. Workspace supplies its artifacts and acceptance evidence
to the shared release capability rather than building a second suite-wide release system.

## Depends on

- `epic:platform/release-automation`
- `epic:platform/launcher-shim`
- `epic:rust-adoption/workspace-migration`
- `epic:rust-adoption/parity-validation`

## What this epic is not

- A prerequisite for local Workspace development or isolated adoption testing.
- Ownership of the platform launcher, its release cache, component downloads, or its source-build selection mechanism.
