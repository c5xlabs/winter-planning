# Scope identity

What a scope is, which names are refused, and the managed variables every computation starts from. The registry is the
one [lock-and-state.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md),
`.winter/state.toml`, holds; the port formula is `port_base(index)` of
[configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
Derived values. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at
`127cdf2`. Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/scope_env.rs — the one module that computes a scope's variables; engineering:scope-computation-authority
pub const WORKSPACE_SCOPE: &str = "workspace";

pub enum Scope { Workspace, Feature(String) }

// The managed variables, before any band entry. Computed from the registry and the configuration only:
// nothing is allocated, probed, or recorded.
pub struct Identity {
    pub scope: Scope,
    pub index: i64,                 // 0 for Workspace; the recorded index for Feature
    pub port_base: Option<i64>,     // None for Workspace, and for a Feature whose port_base overflows (OutsideLayout):
                                    // WINTER_PORT_BASE is not emitted for either
    pub workspace_port_base: i64,   // port_base(0)
    pub service_prefix: String,     // WorkspaceConfig.service_prefix
    pub index_fault: Option<IndexFault>,
}

pub enum IndexFault {        // a recorded index outside the layout; product:runtime-scope-values
    Reserved,                       // 0, the workspace scope's
    Buffer,                         // N + 1, never assigned
    OutsideLayout,                  // below 0, or above envs_per_workspace
}

// Precondition: the workspace loaded (configuration-model.md, Load) and the registry read (state_registry::read).
pub fn identity(
    workspace: &Workspace,
    registry: &BTreeMap<String, i64>,
    scope: &str,
) -> Result<Identity, EnvError>;

/// The managed names, in their order (Interfaces, managed variables); a lookup's environment and the Launch
/// environment of consumers.md remove them.
pub const MANAGED_NAMES: [&str; 5] = ["WINTER_ENV", "WINTER_ENV_INDEX", "WINTER_PORT_BASE",
    "WINTER_WORKSPACE_PORT_BASE", "WINTER_SERVICE_PREFIX"];
/// The managed variables of `identity`, in MANAGED_NAMES order; WINTER_PORT_BASE only for a feature scope.
pub fn managed_variables(identity: &Identity) -> Vec<(String, String)>;
```

Visibility: `identity`, `WORKSPACE_SCOPE`, `Scope`, `Identity`, `IndexFault`, `MANAGED_NAMES`, and `managed_variables`
are `pub` inside `scope_env.rs` and re-exported by name from `lib.rs`, on a `pub use` line of their own, with
`EnvError`, which the scaffold re-exports (`context/architecture/changes.md`, A domain module, of `winter-workspace`),
so the slice that lands each tests it before a command calls it (`context/standards/testing.md`) and the gate finds
nothing unused without an `allow` (`context/standards/rust.md`). `Identity` is re-exported as `ScopeIdentity`, since
`process::Identity` holds `Identity` at the crate root. Everything else here stays `pub(crate)`.

## Interfaces

```text
# reference: modules/workspace/env_provisioner.py:104-182, modules/workspace/env_index.py:133-152,
#            modules/workspace/env_command.py:72-78
identity(scope)
  scope == "workspace"            -> Scope::Workspace, index 0, port_base None, no index_fault
  scope recorded in the registry  -> Scope::Feature(scope), index = the recorded value
                                     port_base = Some(port_base(index))
                                     index_fault = Reserved      when index == 0
                                                   Buffer        when index == len(env_aliases) + 1
                                                   OutsideLayout when index < 0 or index > envs_per_workspace
                                                   None          otherwise
  anything else                   -> EnvError::UnknownScope { scope }
                                     a directory named scope under the root does not make it known; an alias that
                                     ws init has not yet recorded is unknown too
  registered deviation, decision:running-environments/scope-environment/incomplete-launch-refusal: the reference's
  build_env_trio answers an unrecorded name with its alias position or unprobed hash slot, which provisioning, hooks,
  and nested delegation consume; no consumer of identity() derives an index from a name
  every operation that needs a scope's identity calls identity(), provisioning and extension hooks included, so a hook
  or provision step run before ws init records the name is refused with UnknownScope; nested delegation reads the
  index ws init has already recorded (nested-workspaces, delegation.md) and derives none from a name

managed variables, in this order: MANAGED_NAMES, and managed_variables(identity) their values
  WINTER_ENV                   "workspace", or the feature name
  WINTER_ENV_INDEX             index, decimal
  WINTER_PORT_BASE             port_base, decimal; Feature only, and not emitted when port_base overflows
  WINTER_WORKSPACE_PORT_BASE   workspace_port_base, decimal
  WINTER_SERVICE_PREFIX        service_prefix
  these are the base variables of resolution.md; port_base(i) is i64 arithmetic, an overflow is OutsideLayout
  with port_base None; WINTER_ENV still names the feature, so the scope is not taken for the workspace's, and only
  Observe, which goes on under the fault, emits the variables without WINTER_PORT_BASE
```

What each consumer does with an `index_fault` ([consumers.md](./consumers.md) defines the consumer kinds):

| Consumer kind          | Index fault                                                                                                                                |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `Print` (`winter env`) | `EnvError::RecordedIndex { scope, index, fault }`, exit 1                                                                                  |
| `Launch`               | the same error; that scope is not launched                                                                                                 |
| `Observe`              | one warning on stderr naming the scope, the index, and the fault; the recorded index is used, so services started under it are still found |

```rust
// crates/core/src/termination.rs — Termination::Env(Box<EnvError>), exit 1
// (../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md)
#[derive(Debug, thiserror::Error)]
pub enum EnvError {
    UnknownScope { scope: String },                                  // "no env by that name is registered"
    RecordedIndex { scope: String, index: i64, fault: IndexFault },
    Resolution(ResolutionError),                                     // resolution.md; a lookup failure is its Lookup variant
    Interrupted,          // a lookup answered LookupFailure::Interrupted (value-lookups.md, The child); not Termination::Env:
                          // the command file ends it as Termination::Aborted { interrupted: true } (consumers.md)
}
// No variant carries a resolved value or a rendered command (product:private-runtime-values).
```
