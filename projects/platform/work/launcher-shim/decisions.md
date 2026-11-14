# Winter launcher shim decisions

## Version selection (decision:launcher-shim/version-selection)

**Decider:** product owner and engineering

A workspace pins its Workspace release with `versions.workspace` in a `.winterrc` at its root, found as the nearest
`.winterrc` above the caller's directory, and each role's user-wide default lives under `versions` in
`$XDG_CONFIG_HOME/winter/config.toml`; both hold exact release versions only. A root dotfile follows the per-project pin
convention of `.nvmrc` and `.tool-versions` and lets the shim find a pin without parsing Workspace's own configuration,
and the user file sits in XDG config to match `decision:version-cache`.

## Component discovery (decision:launcher-shim/component-discovery)

**Decider:** engineering

The shim runs a component only from the version cache, at `<role>/<version>/<target>/`, after its `component.toml`
confirms role, version, target, and a supported handoff version; it never searches PATH and recognizes no third-party
groups. PATH order is the ambiguity this epic removes, and a manifest check proves compatibility without starting an
extra process.

## Version scope (decision:launcher-shim/version-scope)

**Decider:** product owner and engineering

Each role selects its version independently: a workspace invocation uses the workspace's pin when one applies and the
Workspace role's user-wide default otherwise, while hub and runner invocations use their own role's default regardless
of the current directory, and an unresolved selection is an error rather than a fallback. Hub and runner releases move
on their own schedule rather than with a workspace's pin, and a single version per role per machine is enough until a
per-invocation version or local-build override is added.

## Component boundary (decision:launcher-shim/component-boundary)

The shim launches separate product executables and contains no Workspace or Fleet implementation; its contract is
verified with stand-in executables, without adapting or invoking the current Python Workspace CLI. It extends the
foundation's dev-only sample handoff with product discovery and version selection rather than owning a second sample
implementation.
