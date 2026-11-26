# Consumers

The one entry point every operation that needs a scope's variables calls, what each kind of consumer gets on failure,
when lookups run, and the environment a launched process receives. The service fan-out and the provider invocation that
call it are [provider-contract.md](../../service-orchestration/spec/provider-contract.md). Reference locations are paths
under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`. Message wording is the binary's own
(`decision:functional-parity`).

## Domain

```rust
// crates/core/src/scope_env.rs — engineering:scope-computation-authority, engineering:complete-resolution-outcomes
pub(crate) enum Consumer {
    Print { resolve: bool },   // winter env (env-command.md)
    Launch,                    // service up; any operation that starts a workload with the scope's variables
    Observe,                   // service down, service status, and the teardown computation of ws destroy
}
pub(crate) enum Computed {
    Complete(ScopeEnv),
    // Observe only: the managed variables of scope-identity.md and the failure that stopped the bands
    Degraded { managed: ScopeEnv, cause: EnvError },
}
pub(crate) fn compute(
    ports: &Ports<'_>,
    interrupts: &Interrupts,
    workspace: &Workspace,
    registry: &BTreeMap<String, i64>,
    scope: &str,
    consumer: Consumer,
) -> Result<Computed, EnvError>;
```

## Interfaces

### Failure by consumer

| Consumer            | Lookups                       | `UnknownScope` | Index fault                         | Resolution or lookup failure                                                            |
| ------------------- | ----------------------------- | -------------- | ----------------------------------- | --------------------------------------------------------------------------------------- |
| `Print { resolve }` | `Run` if resolve, else `Mask` | `Err`          | `Err`                               | `Err`                                                                                   |
| `Launch`            | `Run`                         | `Err`          | `Err`                               | `Err`: that scope is not launched                                                       |
| `Observe`           | `Mask`                        | `Err`          | warning; the recorded index is used | `Degraded`: one warning naming the scope and the cause, then the managed variables only |

```text
# product:complete-launch-environment; reference: modules/service/provider_invocation.py:64-92
Launch    registered deviation, decision:running-environments/scope-environment/incomplete-launch-refusal: the
          reference catches a template or shell-reference failure, warns, and launches with no scope variables; only
          a failing command entry stopped it. Here every EnvError stops the scope's launch. The fan-out treats it as
          a failed call that returned 1: no later cell is called, and the cells already called stay reported
          (provider-contract.md, Fan-out, owns the reporting of attempted and unattempted cells)
Observe   registered deviation, product:complete-launch-environment: the reference passes no variables at all
          after such a failure, so a stop or status call lost the scope's identity and port base. Degraded keeps
          the managed variables, and the caller reports the observation as degraded
warnings  name the scope, the band, and the key; never a value (product:private-runtime-values)
```

### When lookups run

```text
# decision:running-environments/scope-environment/lookup-execution-policy
# reference: modules/service/service_fan_out_service.py:115-205, modules/service/service_status_matrix_service.py:46
one dispatch   a consumer computes each scope at most once per dispatch and reuses the result for every call of that
               scope (provider-contract.md, Fan-out). No result is kept across scopes, dispatches, or invocations:
               a workspace-band lookup runs again for each feature scope a dispatch launches
up, start      Launch: lookups run
restart, logs  no scope variables are computed; a provider that needs values runs `winter env SCOPE --resolve` itself,
               so every start and restart gets fresh values
down, status   Observe: no lookup runs, --wait readiness polling included
winter env     only with --resolve
nothing else   no other command of this epic runs a lookup; an operation another epic adds names its Consumer in its
               own spec
```

### Launch environment

```text
# product:complete-launch-environment
the environment of every process started with a scope's variables (provider calls, and lookups through
value-lookups.md) is the environment OsPrograms gives every child, with
  remove   WINTER_ENV, WINTER_ENV_INDEX, WINTER_PORT_BASE, WINTER_WORKSPACE_PORT_BASE, WINTER_SERVICE_PREFIX
  env      the scope's variables (Complete, or Degraded's managed variables), laid over
so a caller that sourced another scope supplies none of its identity, and a workspace-scope process never sees a
WINTER_PORT_BASE. A command entry's key under Mask holds PLACEHOLDER, never a caller's value of that name.
registered deviation, decision:running-environments/scope-environment/inherited-environment: the reference passes
its whole process environment through
```

## Open questions

- [invocation-contract.md](../../../agent-ready-operation/executable-extensions/spec/invocation-contract.md) starts
  every child, hooks and provision handlers included, "from a copy of the binary's own environment", so a caller's stale
  managed variables reach a hook that runs with a scope's variables. The executable-extensions owner says whether the
  `remove` list of the Launch environment applies there.
- What the teardown computation of `ws destroy` does when `Observe` returns `UnknownScope` for an environment selected
  by its directory alone and never recorded
  ([destroy.md](../../../parallel-workspaces/environment-lifecycle/spec/destroy.md), Selection). Environment-lifecycle
  engineering answers with resource-provisioning, once
  `decision:running-environments/resource-provisioning/handler-environment` is made.
