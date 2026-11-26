# Scope environment requirements

For `epic:running-environments/scope-environment`.

## Identify runtime scope (product:runtime-scope-values)

A person can identify the environment, service namespace, and port band a command will use without allocating another
environment. Identity comes from `epic:parallel-workspaces/environment-lifecycle`, not a competing runtime registry.

Acceptance:

- Feature values use the recorded environment identity and index; workspace services use the reserved shared band.
- Workspace scope does not masquerade as a feature environment or emit its band as the feature port base.
- A name that is not a registered environment is refused by every command that computes a scope, and a recorded index
  outside the declared band is reported; neither is silently assigned during inspection.
- Changing declared allocation settings exposes affected assignments; an old assignment is not silently claimed safe.

## Print a scope's environment (product:scope-env-command)

An engineer, or a service provider that `eval`s the output, loads a scope's runtime values with
`winter env SCOPE [--resolve]`, where SCOPE is a registered feature environment or `workspace`. The output is a contract
with those providers, so its shape does not drift.

Acceptance:

- Success prints one `export KEY=<shell-quoted value>` line per key on stdout and exits 0.
- Any failure prints nothing on stdout and exits non-zero, with the reason on stderr.
- A name that is not a registered environment is refused, even when a directory by that name exists.
- Without `--resolve`, a command entry is not run and its key prints the literal placeholder `<unresolved:command>`.

## Resolve layered declarations (product:layered-runtime-values)

The engineer can share workspace values, derive feature values, and override one environment without changing siblings.
Compatibility details are settled by `decision:running-environments/scope-environment/resolution-compatibility`.

Acceptance:

- Effective local configuration is applied before workspace, feature, and named-environment precedence is resolved.
- Workspace-origin values cannot accidentally depend on feature-only values; named overrides affect only their target.
- Supported references and integer offsets work across declaration order; undefined, cyclic, malformed, and blocked
  references identify the responsible entries without returning a partially valid export as complete.
- A value substituted into another entry is inserted as written and never read again as a reference.
- A plain value may redeclare a Winter-managed variable; colliding keys resolve by band rank, then declaration order.
- A declared key that is not a valid shell variable name is refused when configuration loads.

## Inspect without executing lookups (product:offline-env-inspection)

A person can inspect runtime values without running configured external commands, and deliberately request real values
when needed. Masking is not a claim to redact values the person explicitly resolves.

Acceptance:

- Default inspection, teardown computation, and status/readiness polling do not execute configured value lookups.
- Unresolved command-derived values are identifiable; references to unexecuted multi-value imports never silently become
  empty strings or valid credentials.
- Explicit resolution produces shell-safe sourceable values only after the whole requested computation succeeds.
- Inspection failure emits no partial sourceable result.

## Obtain external runtime values (product:external-value-resolution)

The engineer can use their own value-producing program without Winter bundling its secrets tool or requiring a language
for that program. When lookups run is settled by
`decision:running-environments/scope-environment/lookup-execution-policy`.

Acceptance:

- Single-value output and multi-value dotenv or flat JSON imports remain available, with an output-key allow-list.
- Anonymous import labels are not exported; command output cannot replace Winter-managed variables.
- Lookups run once per scope each time services start or restart, so a restart obtains fresh values; a lookup shared by
  several scopes runs for each. Inspection, status, and stopping never run them.
- Failure, invalid output, missing executable, or timeout names the entry and fails the requested resolution rather than
  launching with an omitted or placeholder value.
- Each lookup runs from the workspace root with closed stdin and a 30-second deadline; a timed-out lookup leaves no
  process it started running.

## Refuse incomplete launch environments (product:complete-launch-environment)

An agent learns that a required environment could not be computed before a workload is launched unconfigured, under
`decision:running-environments/scope-environment/incomplete-launch-refusal` and
`decision:running-environments/scope-environment/inherited-environment`.

Acceptance:

- A failed required value or template computation prevents the affected launch; a diagnostic alone is not success.
- Previously launched targets are identified if a later scope fails, with remaining targets marked not attempted.
- Teardown and inspection retain independently known target identity even when optional runtime values cannot resolve;
  degraded observation is identified, not represented as verified readiness.
- Caller values do not silently supply another scope's Winter identity or an explicitly unresolved required key.

## Protect resolved values (product:private-runtime-values)

An engineer can use external credentials without Winter copying their resolved values into routine diagnostics or
persistent files. What is exposed is settled by
`decision:running-environments/scope-environment/persistent-value-exposure`.

Acceptance:

- Errors identify the declared entry and never print a resolved value, including a value substituted into another
  entry's command.
- Import parse errors do not print secret-bearing output lines; routine traces do not capture resolved environments.
- Winter writes no resolved value to disk; it is printed only by `--resolve` and passed only to launched workloads.
- A failing lookup's stderr is shown in its error, and masking does not promise protection from a user program printing
  its own secrets.
- Shell opt-in states its interpolation limits rather than claiming that environment-based shell expansion is prevented.
