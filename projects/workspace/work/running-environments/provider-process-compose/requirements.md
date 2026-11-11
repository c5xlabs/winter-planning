# Process-compose provider requirements

For `epic:running-environments/provider-process-compose`.

## Operate isolated supervisor instances (product:process-compose-instances)

An engineer can operate feature processes and workspace singletons through a native first-party provider without Python.

Acceptance:

- Control, declaration translation, status, logs, diagnostics, and destroy integration need no first-party Python; a
  supported process-compose binary remains a disclosed prerequisite.
- Each scope has its own instance identity, endpoint, state, and logs; long paths do not make one scope fall back onto
  another scope's shared endpoint.
- Repeated up preserves unchanged running process identities, applies changed definitions with stated restart effects,
  and can restart stopped selected processes.
- Missing environment inspection creates no directories; up refuses a missing root rather than reconstructing it.
- Conflicting mutations cannot silently overwrite each other, and unrelated scopes remain independently usable.

## Launch selected process dependencies (product:process-compose-launch)

A person can start selected processes and their declared local prerequisites with predictable runtime values and health.

Acceptance:

- Partial up includes the transitive supported same-scope dependencies and keeps unrelated running processes enabled;
  dependency ordering does not depend on declaration/map order or first-update quirks.
- Working directories, service mappings, literal/relative ports, and command aliases have explicit compatibility;
  declared ports remain observations of intent, not a guarantee of binding or band capacity.
- Each actual launch or restart follows the agreed scope-value refresh policy; generated configuration does not expose
  resolved values beyond the explicitly chosen persistence policy.
- URL and command readiness probes, startup retries, and restart policies remain supported; probe environment and
  substitution behavior are stated rather than presumed identical to tmux.
- Startup waiting has a finite caller-aligned budget, identifies unready services, and fails early for terminal failures
  within its promised coverage; unprobed process handling follows the common readiness decision.

## Stop and recover owned processes (product:process-compose-recovery)

The engineer can stop a scope or recover a dead daemon without terminating a reused identity or a neighbor's processes.

Acceptance:

- Whole shutdown stops the owned instance and processes; partial shutdown/restart affects only matching processes and
  leaves unrelated processes and the daemon running.
- Absent instances are safe shutdown no-ops; stale instances and live-but-unresponsive daemons are distinguished, with
  ownership and bounded escalation required before orphan cleanup.
- Missing binary cleanup is possible where ownership-safe stale recovery does not require the supervisor; otherwise the
  operation states what could not be stopped instead of hiding failure.
- A daemon outliving its environment root is either recoverable by stable owned identity without recreating the root or
  explicitly refused with guidance; unlocked fallback behavior is not silently presented as concurrency-safe.
- Process/session reuse, lost state, and unreadable state cannot authorize signaling unrelated processes; unsupported OS
  recovery follows `decision:running-environments/provider-process-compose/recovery-platforms`.

## Observe supervised processes and logs (product:process-compose-observation)

A person can inspect stopped or live processes and read their output even after the daemon exits.

Acceptance:

- State, readiness, process handle, start time, ports, and log location have explicit known/unknown mappings; stale or
  unreachable control endpoints produce diagnosed incomplete observation rather than false readiness.
- Startup-contributed live services and their logs remain discoverable for later supported actions under the common
  declaration-continuity policy.
- File logs carry attributable timestamps, support inclusive time bounds and tail selection, and expose restart
  truncation and rotation guarantees; follow resumes correctly after supported truncation/replacement.
- Every selected service and scope receives fair live log delivery; a sequential never-ending first-scope follow is not
  acceptable, and stopping follow leaves supervised processes running.

## Refuse unsupported runtime features (product:process-compose-feature-limits)

An author can tell which parts of a portable service declaration this provider actually honors before launch.

Acceptance:

- Interactive empty commands, log/uptime probes, and cross-scope/provider dependencies are either deliberately supported
  or explicitly refused; accepting a declaration must not silently omit its required behavior.
- Tmux-only pane/log fields are identified as non-applicable; startup and native restart policy precedence is explicit.
- Image-only startup contributions and name collisions are diagnosed rather than accepted as runnable process commands.
- Minimum supported supervisor version, endpoint-length limits, platform restrictions, and version-specific update/probe
  behavior are discoverable under `decision:running-environments/provider-process-compose/supervisor-compatibility`.
- Scope wildcard selection is supported through the common interface or refused before mutation, not silently dropped.
