---
epic: release-automation
refinement: outline
status: planned
---

# Winter release automation

Provides shared release enablement for Platform, Workspace, and Fleet under `decision:shared-release-capability`; its
product requirements live in [requirements.md](./requirements.md).

## The problem

A working development build is not yet a repeatably distributed product. Across Platform, Workspace, and Fleet,
maintainers need a common way to identify, verify, and publish releases without inventing separate policies and release
machinery for every executable, service, or web application.

Release readiness must not become a prerequisite for rapid foundation and launcher development. Packaging, signing,
publication credentials, and release-policy choices have their own delivery boundary
(`decision:release-delivery-separation`).

## The outcome

Maintainers can adopt the same governed release lifecycle across all three projects, with each product retaining its
artifact-specific packaging and verification integration. The shared capability exercises the release governance through
reusable tooling and procedures rather than maintaining it only as prose.

Rust CLI packaging uses `cargo-dist` under `decision:release-automation/release-automation`; service/container and web
artifacts follow the same lifecycle through appropriate packaging tools. The platform executable is the first consumer,
with verified archives and Homebrew installation that exclude the development sample, not a prerequisite for forcing
every product to release simultaneously.

Consumer release work declares a dependency on `epic:platform/release-automation` through its epic's `## Depends on` or
release milestone's chart. Feature-development work does not acquire that dependency merely because its project will
eventually release through the shared capability.

## What this epic is not

- A prerequisite for completing the foundation or implementing the launcher shim.
- Workspace or Fleet feature implementation, product dispatch, component downloads, or self-update.
- Enforcing a release train or published-release compatibility promises on unreleased development builds.
- Migrating every existing Blizzard release workflow or imposing its full cadence and support windows on the suite.

## Depends on

- `epic:cli-foundation` for the executable and its normal production build and verification commands.
