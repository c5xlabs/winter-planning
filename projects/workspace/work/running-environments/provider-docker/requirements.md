# Docker provider requirements

For `epic:running-environments/provider-docker`.

## Operate isolated compose scopes (product:docker-compose-scopes)

An engineer can run feature containers and workspace singletons through a native first-party provider without Python.

Acceptance:

- Control, diagnostics, declaration handling, startup checks, logs, and destroy integration have no first-party Python
  requirement; Docker daemon access and supported Compose v2 remain explicit prerequisites.
- Feature and workspace resources use distinct workspace-owned project namespaces; optional provider namespace overrides
  cannot silently disagree between startup, status, restart, logs, and teardown.
- Scope-pure compose selection prevents whole-scope startup from starting another scope's services; missing or obsolete
  configuration is diagnosed consistently by every action.
- Relative compose paths are interpreted against their declared configuration location, not an incidental caller cwd.
- An intentionally empty scope is distinguishable from a missing prerequisite or failed declaration load.

## Allocate and explain published ports (product:docker-port-values)

The engineer can run several environments with distinct published ports and inspect the actual published endpoints.

Acceptance:

- Feature substitution values derive from the assigned feature band; workspace services use the shared band explicitly.
- Declaration-order offsets, service-name transformations, limits on band capacity, and changes caused by reordering are
  disclosed under `decision:running-environments/provider-docker/port-assignment`.
- Status reports known actual published host ports, not an unverified substitution value presented as a binding.
- Missing scope values fail affected launch; restart/logs of existing resources do not demand fresh secrets merely to
  interpolate unused launch-time fields.

## Preserve targeted container effects (product:docker-targeted-lifecycle)

A person can start, stop, or restart selected containers knowing which dependencies and durable resources are affected.

Acceptance:

- Partial startup targets selected services and exposes dependency side effects; partial shutdown stops/removes only
  selected containers without tearing down unrelated project networks or containers.
- Whole shutdown removes its containers/networks but retains named volumes unless a separate deliberate destructive
  resource operation authorizes removal.
- Repeated startup reconciles existing resources according to the provider's declared behavior; failed commands retain
  earlier effects and failures instead of implying rollback.
- Destroy uses the same current scope, prefix, and compose selection as ordinary shutdown; a legacy shell parser cannot
  skip cleanup and still claim verified success.
- Restart and logs support cross-environment selection or explicitly refuse it before effects; they do not silently
  discard wildcard scopes.

## Observe readiness and logs (product:docker-health-and-logs)

The engineer can wait for container readiness and read every selected service's output through the common interface.

Acceptance:

- Running/healthy, running/unhealthy, starting, exited, absent, and unprobed containers have explicit state/readiness
  meanings; startup does not treat starting or exited containers as ready.
- Native startup waiting honors the agreed caller budget and selected target coverage; dependency and replica coverage
  follow `decision:running-environments/provider-docker/readiness-coverage` rather than silently using the first visible
  container as proof of all services.
- Failed daemon/compose observation is an unavailable result, not a valid stopped document returned as success.
- Log time bounds and tail options reach the backend where supported, with stable timestamp attribution and common
  backstops; following several services is concurrent/fair, not an endless first-service drain.
- Canceling followers cleans up command-owned backend processes while containers continue running.

## Expose provider capability gaps (product:docker-capability-limits)

An author learns whether Docker can honor a declaration before relying on it for setup or service operation.

Acceptance:

- Supported catalog and ownership cover configured feature/workspace services, including known-stopped targets.
- Startup-contributed service declarations are either supported through later lifecycle actions or explicitly refused;
  image/cmd/target fields are not silently treated as a complete compose definition.
- Missing declaration entries, backend-only services, multiple replicas, and unsupported shared-manifest features expose
  their routing/observation limits under `decision:running-environments/provider-docker/service-declaration-support`.
- Runtime and operating-system prerequisites, and unsupported capabilities on a supported platform, are discoverable
  without starting containers.
