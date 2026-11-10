# Winter launcher shim decisions

## Component discovery (decision:launcher-shim/component-discovery)

**Decider:** engineering

**Open.** How installed components are registered or discovered, how an executable proves its role and protocol
compatibility, and which sources take precedence; this blocks `product:reach-component` and
`product:explain-dispatch-failure`. Decide explicitly whether PATH discovery and third-party top-level groups belong in
the first contract rather than executing every matching name by default.

## Version selection (decision:launcher-shim/version-selection)

**Decider:** product owner and engineering

**Open.** The component pin schema, how the relevant workspace is found, and how an unpinned invocation selects a
version inside or outside a workspace; this blocks `product:respect-component-version`. Component identity, release
version, and target platform must remain distinct in the cache without treating the shim's version as the product's.

## Component boundary (decision:launcher-shim/component-boundary)

The shim launches separate product executables and contains no Workspace or Fleet implementation; its contract is
verified with stand-in executables, without adapting or invoking the current Python Workspace CLI. It extends the
foundation's dev-only sample handoff with product discovery and version selection rather than owning a second sample
implementation.
