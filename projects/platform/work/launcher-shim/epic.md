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
Hub and Runner also need directory-scoped version selections without adopting Workspace's configuration or requiring a
Winter workspace.
Replacing the installed singleton must not break existing Python workspaces or require their immediate adoption of the
unfinished Rust Workspace product.

## The outcome

The `winter` command carries the person or agent to a separately installed product executable while preserving the
invocation they intended. The platform keeps its own commands; Workspace and Fleet keep their implementations and can be
installed independently. The shim follows `decision:launcher-language` and is added to the foundation executable, not
shipped as a competing command.
Workspace owns its pin in `.winter/config.toml`; platform-owned `.winterrc` files supply inherited per-role selections
from filesystem root to the caller's directory, with child values overriding ancestor values and user-selected defaults
available when no directory selection applies.

Under `decision:singleton-launcher-succession`, the same executable preserves opt-in legacy commands by launching the
workspace's existing Python application, retains agents' first-argument `--winter=PATH` implementation override without
changing the target workspace, and temporarily retains the old `winter init` bootstrap. Explicit product groups always
select Rust components, so launcher replacement does not imply workspace adoption or Rust parity.

This epic extends the development-sample process boundary owned by `epic:cli-foundation` with installed-component
discovery, compatibility checks, version selection, and bounded legacy launch preparation. Both dispatch paths are
verified with controlled stand-in executables and isolated filesystems (`decision:launcher-shim/component-boundary`).

## Left for later

- Fetching published component releases into the version cache: `epic:component-install`.

## What this epic is not

- A port of Python Workspace business logic, legacy aliases into Rust Workspace, or Python fallback from modern groups.
- Workspace operations, Fleet business logic, or starting and supervising Fleet services.
- Self-update or release publishing. Missing Rust components are reported, not installed; legacy Python dependency
  synchronization is a separate compatibility behavior.
- Cutover documentation, automated singleton installation, or rollback tooling; replacement is a manual operator action
  under `decision:launcher-shim/manual-cutover`.
- Building Rust components from source or introducing a Rust development-source override.
- Choosing a Rust release through a per-invocation flag or environment override. Directory-scoped selections and the
  existing Python-only `--winter=PATH` source override are in scope.
- Adding `--env` or Rust build-and-run conveniences; local Rust development invokes sub-executables directly under
  `decision:workspace/development-component-selection`.
- Compatibility between hub and runner versions, and the identity of separate hub or runner instances; those belong to
  Fleet.

## Depends on

- `epic:cli-foundation`.
