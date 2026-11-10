# Milestones

Where the platform is headed, as destinations in the work of the people it serves, in the order they are meant to be
reached. Each milestone names the epics needed to reach it; each epic's own status is the truth, so this is a plan, not
a status board. Once a milestone is reached, it moves to the [delivered ledger](./delivered.md).

## Inspect Winter before setup (milestone:inspect-before-setup)

A person can ask the platform executable how to use it and which version it is, without a workspace, a fleet, or an
existing Winter installation. Unsupported invocations explain the boundary instead of handing off to another tool.
A maintainer can verify and publish this foundation, and a person can install the released executable through Homebrew
without setting up a compiler; contributors and reviewers share documented architectural rules and test methods.
Contributors can also run a dev-only Rust sample through the platform CLI to observe the separate-process boundary;
released installations do not include that sample.

| Epic | Status |
| --- | --- |
| `epic:cli-foundation` | planned |

## Reach independent products (milestone:reach-independent-products)

A person can use the same entry point for independently installed Winter products, with the component version selected
for their workspace. Missing components are explained without running a different implementation or changing their
setup.

| Epic | Status |
| --- | --- |
| `epic:cli-foundation` | planned |
| `epic:launcher-shim` | planned |
