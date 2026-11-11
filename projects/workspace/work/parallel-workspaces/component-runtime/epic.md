---
epic: parallel-workspaces/component-runtime
refinement: outline
status: planned
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

## What this epic is not

- An in-process plugin in the platform executable or a copy of the platform implementation.
- A bridge that delegates unimplemented commands to Python.
