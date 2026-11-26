---
name: scope-identity
depends-on:
  - module-scaffold
---

# A scope's identity from the registry

Adds `identity`, `Scope`, `Identity`, `WORKSPACE_SCOPE`, and the managed variables of
[scope-identity.md](../spec/scope-identity.md): the scope a name addresses, its recorded index and port bases, the
`IndexFault` of a recorded index the layout does not place in the band, and the five managed variables in their order.
The registry is the one `state_registry::read` gives, which `epic:parallel-workspaces/environment-lifecycle` writes;
nothing here allocates, probes, or records. Delivers the identity bullets of `product:runtime-scope-values` and
`engineering:scope-computation-authority`'s refusal to derive an index from a name, under
`decision:running-environments/scope-environment/incomplete-launch-refusal`, a registered deviation.

## Change

- `scope_env.rs` gains `identity`: `workspace` is `Scope::Workspace` with index 0 and no port base; a recorded name is
  `Scope::Feature` with the recorded index, `port_base(index)` of configuration-model.md, Derived values, in checked
  `i64` arithmetic, and its `IndexFault`; anything else is `EnvError::UnknownScope`.
- `managed_variables` builds the managed variables, `WINTER_ENV`, `WINTER_ENV_INDEX`, `WINTER_PORT_BASE` (feature only),
  `WINTER_WORKSPACE_PORT_BASE`, and `WINTER_SERVICE_PREFIX`, in that order from an `Identity`; the five names are
  `MANAGED_NAMES`, the one list that `run-lookup` removes from a lookup's environment and the Launch environment of
  [consumers.md](../spec/consumers.md) names.
- `identity`, `WORKSPACE_SCOPE`, `Scope`, `MANAGED_NAMES`, and `managed_variables` are re-exported by name from `lib.rs`
  for its tests, and `Identity` as `ScopeIdentity`, since `process::Identity` holds `Identity` at the crate root
  ([scope-identity.md](../spec/scope-identity.md), Domain); no command calls them yet.

## Acceptance

- Unit tests over a loaded workspace: `workspace` answers index 0, no port base, and `port_base(0)` as the workspace
  base; a recorded `alpha` at 1 answers its port base and no fault.
- Unit tests of `IndexFault`: a recorded 0 is `Reserved`, `len(env_aliases) + 1` is `Buffer`, a negative index and one
  above `envs_per_workspace` are `OutsideLayout`, and an index whose port base overflows `i64` is `OutsideLayout`.
- Unit tests: an unrecorded name is `UnknownScope` when `<root>/<name>` is a directory, and when it is an alias that
  `ws init` has not recorded.
- Unit tests of `managed_variables`: a feature scope gives the five of `MANAGED_NAMES` in order; the workspace scope
  gives four, with no `WINTER_PORT_BASE`.
- `cargo dev gate slice` exits 0.

## Out of scope

- What each consumer does with an `IndexFault` lands with `compute-print` and `launch-observe-consumers`.
- The alias-slot and shared-index findings are reported by `ws list` of `epic:parallel-workspaces/environment-lifecycle`
  (`product:parallel-workspaces/environment-lifecycle/environment-state-recovery`).
- Provisioning, hooks, and nested delegation call `identity` in the epics that own them.
