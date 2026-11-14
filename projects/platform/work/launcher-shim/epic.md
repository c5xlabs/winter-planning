---
epic: launcher-shim
refinement: outline
status: planned
---

# Winter launcher shim

A standalone capability epic supporting the Workspace and Fleet entry points; its product requirements live in
[requirements.md](./requirements.md).

## The problem

A person using independently installed Winter products should not have to remember a different executable and version
selection procedure for each. In two workspaces pinned to different versions, a command must reach the selected product
version rather than whichever binary happens to come first on PATH, and the hub and runner a person runs must be able to
move to a new release without waiting on every workspace's pin.

## The outcome

The `winter` command carries the person or agent to a separately installed product executable while preserving the
invocation they intended. The platform keeps its own commands; Workspace and Fleet keep their implementations and can be
installed independently. The shim follows `decision:launcher-language` and is added to the foundation executable, not
shipped as a competing command.

This epic extends the development-sample process boundary owned by `epic:cli-foundation` with installed-component
discovery, compatibility checks, and version selection. Product dispatch is verified with controlled stand-in
executables, not by connecting to the current Python Workspace CLI (`decision:launcher-shim/component-boundary`).

## What this epic is not

- A port, wrapper, fallback, or compatibility layer for the existing Python Workspace CLI.
- Workspace operations, Fleet business logic, or starting and supervising Fleet services.
- Component downloads, installers, self-update, or release publishing. Missing components are reported, not installed.
- Building components from source or introducing a development-source override.
- Choosing a version for a single invocation, or running more than one version of the same role side by side on one
  machine outside workspace pins.
- Compatibility between hub and runner versions, and the identity of separate hub or runner instances; those belong to
  Fleet.

## Depends on

- `epic:cli-foundation`.
