# Winter launcher shim decisions

## Manual singleton replacement (decision:launcher-shim/manual-cutover)

**Decider:** product owner

Singleton installation and rollback are manual operator actions; this epic requires neither cutover documentation nor
migration tooling because the operator owns the affected installations. Dispatch compatibility remains separate from
Rust Workspace adoption and its capability, provider, and migration gates.

## Routing boundary (decision:launcher-shim/routing-boundary)

**Decider:** product owner and engineering

Modern groups, bootstrap, and platform controls are recognized only at the leading platform position, after the optional
initial Python source override and root verbosity. After an opaque Python option prefix begins, only defined legacy
names select flag-gated forwarding, so option values cannot activate Rust dispatch, bootstrap, or platform controls.

## Version selection (decision:launcher-shim/version-selection)

**Decider:** product owner and engineering

A workspace owns its Workspace pin as `versions.workspace` in `.winter/config.toml`; platform-owned `.winterrc` files
can live in any directory and supply exact per-role versions, merged from filesystem root through the caller's directory
with child values overriding only the keys they set, and platform-owned `$XDG_CONFIG_HOME/winter/config.toml` records
each role's user-selected default under `versions`. This separates Workspace configuration from platform selection:
Hub and Runner never read `.winter/config.toml`, and `~/.winterrc` participates only when it is on the ancestor path,
not as the user-wide default file.

## Component discovery (decision:launcher-shim/component-discovery)

**Decider:** engineering

The modern shim path runs a component only from the version cache, at `<role>/<version>/<target>/`, after its
`component.toml` confirms role, version, target, and a supported handoff version; it never searches PATH or recognizes
third-party groups. PATH order is the ambiguity this epic removes, and a manifest check proves compatibility without
starting an extra process.

## Version scope (decision:launcher-shim/version-scope)

**Decider:** product owner and engineering

Each role selects independently: Workspace uses the nearest `.winter/config.toml` pin when present and the inherited
`.winterrc` Workspace selection otherwise, while Hub and Runner use their inherited `.winterrc` selections, with each
role falling back to its user-selected default only when no applicable directory selection exists. Directory-scoped
selections let deployments move components independently without requiring a Winter workspace; a missing selected
release or unresolved selection is an error, never a fallback to a different cached version or a PATH executable.

## Component boundary (decision:launcher-shim/component-boundary)

The shim launches separate product executables and contains no Rust Workspace or Fleet implementation; the bounded
Python-launch preparation, legacy `--winter=PATH` source override, and temporary bootstrap follow
`decision:singleton-launcher-succession`, not a fallback from modern dispatch. Verification uses stand-in Rust and
Python entrypoints and a stub dependency synchronizer rather than the installed launcher or live products, so
compatibility testing cannot mutate a real workspace.
