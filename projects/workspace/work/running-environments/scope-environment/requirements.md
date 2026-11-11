# Scope environment requirements

For `epic:running-environments/scope-environment`.

## Identify runtime scope (product:runtime-scope-values)

A person can identify the environment, service namespace, and port band a command will use without allocating another
environment. Identity comes from `epic:parallel-workspaces/environment-lifecycle`, not a competing runtime registry.

Acceptance:

- Feature values use the recorded environment identity and index; workspace services use the reserved shared band.
- Workspace scope does not masquerade as a feature environment or emit its band as the feature port base.
- Unknown or inconsistent identity is diagnosed rather than silently assigned during inspection.
- Changing declared allocation settings exposes affected assignments; an old assignment is not silently claimed safe.

## Resolve layered declarations (product:layered-runtime-values)

The engineer can share workspace values, derive feature values, and override one environment without changing siblings.
Compatibility details are settled by `decision:running-environments/scope-environment/resolution-compatibility`.

Acceptance:

- Effective local configuration is applied before workspace, feature, and named-environment precedence is resolved.
- Workspace-origin values cannot accidentally depend on feature-only values; named overrides affect only their target.
- Supported references and integer offsets work across declaration order; undefined, cyclic, malformed, and blocked
  references identify the responsible entries without returning a partially valid export as complete.
- A dormant named-environment override remains inspectable and distinguishable from an active override.
- Redefinitions of managed identity variables and imported-key collisions follow an explicit rule visible to the author.

## Inspect without executing lookups (product:offline-env-inspection)

A person can inspect runtime values without running configured external commands, and deliberately request real values
when needed. Masking is not a claim to redact values the person explicitly resolves.

Acceptance:

- Default inspection, teardown computation, and status/readiness polling do not execute configured value lookups.
- Unresolved command-derived values are identifiable; references to unexecuted multi-value imports never silently become
  empty strings or valid credentials.
- Explicit resolution produces shell-safe sourceable values only after the whole requested computation succeeds.
- Inspection failure emits no partial sourceable result, and explicit resolution warns that real values are disclosed.

## Obtain external runtime values (product:external-value-resolution)

The engineer can use their own value-producing program without Winter bundling its secrets tool or requiring a language
for that program. Lookup execution and refresh frequency are explicit under
`decision:running-environments/scope-environment/lookup-execution-policy`.

Acceptance:

- Single-value output and multi-value dotenv or flat JSON imports remain available, with an output-key allow-list.
- Anonymous import labels are not exported; command output cannot replace Winter-managed variables.
- Each operation states when lookups run, how often each can run, and whether restart obtains fresh values; ordinary
  polling does not repeat them.
- Failure, invalid output, missing executable, or timeout names the entry and fails the requested resolution rather than
  launching with an omitted or placeholder value.
- The execution baseline is a 30-second deadline per lookup with closed stdin; a longer or interactive policy requires
  an explicit decision, not an unnoticed migration change.

## Refuse incomplete launch environments (product:complete-launch-environment)

An agent learns that a required environment could not be computed before a workload is launched unconfigured.

Acceptance:

- A failed required value or template computation prevents the affected launch; a diagnostic alone is not success.
- Previously launched targets are identified if a later scope fails, with remaining targets marked not attempted.
- Teardown and inspection retain independently known target identity even when optional runtime values cannot resolve;
  degraded observation is identified, not represented as verified readiness.
- Caller values do not silently supply another scope's Winter identity or an explicitly unresolved required key.

## Protect resolved values (product:private-runtime-values)

An engineer can use external credentials without Winter copying their resolved values into routine diagnostics or
unexpected persistent files.

Acceptance:

- Errors identify the declared lookup rather than echoing a rendered command containing another entry's value.
- Import parse errors do not print secret-bearing output lines; routine traces do not capture resolved environments.
- Any child stderr exposure and any persistence needed by a provider are disclosed; masking does not promise protection
  from a user program printing its own secrets.
- Shell opt-in states its interpolation limits rather than claiming that environment-based shell expansion is prevented.
