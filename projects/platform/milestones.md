# Milestones

Where the platform is headed, as destinations in the work of the people it serves, in the order they are meant to be
reached. Each milestone names the epics needed to reach it; each epic's own status is the truth, so this is a plan, not
a status board. Once a milestone is reached, it moves to the [delivered ledger](./delivered.md).

## Winter shim (milestone:winter-shim)

A person can install the published `winter` executable through Homebrew, fetch the Workspace, hub, or runner releases
they select, and run them through that one entry point without building them or arranging files by hand. Existing Python
workspaces can keep using their implementation through explicit opt-in, without adopting Rust Workspace. Missing
components are explained without falling back to another implementation; fetching verifies releases without changing
version selections or leaving a failed installation usable. Maintainers can build, verify, and publish the launcher
through a repeatable release process.

| Epic                      | Status    |
| ------------------------- | --------- |
| `epic:cli-foundation`     | delivered |
| `epic:launcher-shim`      | ready     |
| `epic:release-automation` | planned   |
| `epic:component-install`  | planned   |
