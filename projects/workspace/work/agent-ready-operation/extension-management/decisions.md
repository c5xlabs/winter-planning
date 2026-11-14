# Extension management decisions

## Adoption and enablement policy (decision:agent-ready-operation/extension-management/adoption-policy)

**Decider:** product owner and Workspace engineering

Adoption is one workspace-wide mode, `adopt_extensions` (`winter` by default, `all`, or `none`), plus
`extension =
false` on a repository entry, as in the reference tool. The mode gates projection, hooks, context, and
checks; it does not gate the dependency graph or capability lookup.

## Activation state and concurrent changes (decision:agent-ready-operation/extension-management/activation-ownership)

**Decider:** Workspace engineering and the product owner

No activation record is kept, as in the reference tool: each `ws init` recomputes the adopted set from configuration and
prunes projected artifacts by live prefix. `ws prune` removes the copies of extensions no longer declared.

## Manifest compatibility (decision:agent-ready-operation/extension-management/manifest-policy)

**Decider:** Workspace and extension-contract engineering

`winter-ext.toml` keeps the reference tool's format: unknown keys are ignored, the legacy `orchestrate_services` alias
is read, and each kind of invalid content fails at the scope the extension manifest spec states.
