# Delivered

The record of the destinations Winter Workspace has reached, newest first, each with the epics that carried it there. It
is a record of what people can now do, not a status board.

## Declare a workspace without forking Winter (milestone:declare-workspace)

An engineer can create and inspect a workspace, declare its repositories and local settings, and give an agent guidance
that matches the selected tool. They do not clone Winter's framework source to start configuring their own system.

| Epic                                               | Notes                                                                                                        |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `epic:parallel-workspaces/component-runtime`       | The Rust Workspace component, invoked directly or through `winter workspace`, with consistent results        |
| `epic:rust-adoption/parity-validation`             | Recorded reference scenarios that replay the replacement against the Python tool                             |
| `epic:parallel-workspaces/workspace-configuration` | Workspace discovery and shared and local settings read as the reference does, invalid ones refused           |
| `epic:parallel-workspaces/repository-catalog`      | Listing and editing the declared repository inventory, apart from its checkouts (#196)                       |
| `epic:parallel-workspaces/reproducible-inputs`     | Standalone refs resolved, updated, frozen, and fingerprinted without changing working state (#213)           |
| `epic:agent-ready-operation/context-navigation`    | Topic guidance from the component, with the safety rules reaching the agent first (#236)                     |
| `epic:parallel-workspaces/workspace-bootstrap`     | `init`, `ws init`, and `ws prune`: create a workspace, reconcile its checkouts, remove stale material (#245) |
