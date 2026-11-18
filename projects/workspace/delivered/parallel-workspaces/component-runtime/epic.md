---
epic: parallel-workspaces/component-runtime
refinement: outline
status: delivered
---

# Invoke Workspace as a product component

Part of `initiative:parallel-workspaces`; product requirements live in [requirements.md](./requirements.md). Supports
the other parity epics through a shared, independently callable command boundary.

## The problem

Engineers and agents need to know which Workspace component they are invoking and whether its command succeeded. The old
source-local launcher cannot be the foundation of an independently installed Rust product.

## The outcome

The engineer can invoke the Rust `winter-workspace` component directly or through `winter workspace`, obtain truthful
help and version information without a workspace, and use commands whose refusals and machine-readable results are
consistent. Maintainers have a documented and checked development foundation before adding destructive operations.

## Depends on

- `epic:platform/cli-foundation`

## Left for later

Installed-component selection and handoff belong to `epic:platform/launcher-shim`; publishing Workspace belongs to
`epic:rust-adoption/workspace-release`. Direct local development does not wait for either release delivery or downloads.

Three product requirements are delivered here only in part, because the rest of each is a property of commands this epic
does not build:

- `product:automation-boundary`: this epic delivers usage errors that start nothing, plain-text errors, and the exit
  codes. The confirmation prompt and the options that skip it are delivered with the first command that prompts, in
  `epic:parallel-workspaces/environment-lifecycle`; the structured result of each command is delivered by the epic that
  builds the command.
- `product:discover-command-scope`: this epic delivers help that lists only the commands the binary has. Help that names
  a command's required scope and its destructive effect is delivered by the epic that builds each command.
- `product:local-component-evaluation`: this epic delivers a locally built binary that is started by its path and is
  never installed over a release. Selecting that binary for an intended workspace is
  `decision:development-component-selection`, delivered by `epic:platform/launcher-shim`.

## What this epic is not

- An in-process plugin in the platform executable or a copy of the platform implementation.
- A bridge that delegates unimplemented commands to Python.
