# Milestones

Where the platform is headed, as destinations in the work of the people it serves, in the order they are meant to be
reached. Each milestone names the epics needed to reach it; each epic's own status is the truth, so this is a plan, not
a status board. Once a milestone is reached, it moves to the [delivered ledger](./delivered.md).

## Reach independent products (milestone:reach-independent-products)

A person can use the same entry point for independently installed Winter products, with the component version selected
for their workspace. Missing components are explained without running a different implementation or changing their
setup.

| Epic                  | Status    |
| --------------------- | --------- |
| `epic:cli-foundation` | delivered |
| `epic:launcher-shim`  | planned   |

## Install a published platform release (milestone:install-platform-release)

A maintainer can verify and publish an identifiable platform release, and a person can install its prebuilt executable
through Homebrew without setting up a compiler. Published artifacts exclude the development sample, and the release
procedure applies the shared package and versioning governance without blocking ongoing foundation or shim development.

| Epic                      | Status    |
| ------------------------- | --------- |
| `epic:cli-foundation`     | delivered |
| `epic:release-automation` | planned   |

## Install released components (milestone:install-released-components)

A person can fetch the published Workspace, hub, or runner release their workspace or machine selects, and then run it
through the same `winter` entry point, without building it or arranging files by hand. A fetch that fails or does not
verify leaves nothing half-installed, and fetching never changes which version anything selects.

| Epic                      | Status  |
| ------------------------- | ------- |
| `epic:launcher-shim`      | planned |
| `epic:release-automation` | planned |
| `epic:component-install`  | planned |
