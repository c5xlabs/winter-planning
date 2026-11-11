# Tmux provider requirements

For `epic:running-environments/provider-tmux`.

## Run isolated terminal services (product:tmux-terminal-services)

A person can operate feature and shared services in tmux without Python in the supported first-party workflow.

Acceptance:

- Control, observation, log capture, convenience entry points, diagnostics, and teardown do not invoke first-party
  Python; user programs retain their own runtime choices under `decision:rust-replacement`.
- Workspace and feature sessions have distinct ownership and namespaces; another scope's values do not leak through the
  shared tmux server environment or an inherited caller environment.
- Interactive empty-command services, explicit pane routing, optional layout hooks, and attach remain available; missing
  tmux or a required login-shell prerequisite is diagnosed without suggesting Winter replaces that prerequisite.
- Repeated whole-scope and partial startup have explicit reconciliation behavior under
  `decision:running-environments/provider-tmux/session-reconciliation`; partial requests never touch unrelated panes.

## Launch with declared environment and ordering (product:tmux-launch-behavior)

The engineer can launch selected services with their working directory, environment mapping, and prerequisites resolved
before destructive changes.

Acceptance:

- Relative service working directories remain inside their scope; scope values, optional machine-local environment
  source, and ordered service mappings have declared precedence, shared by launch and health evaluation.
- Invalid selected mappings, missing required source files, and unusable pane targets fail before the affected launch or
  restart mutates panes; unrelated already-running services are not blocked by unselected mappings.
- Local dependency ordering and cross-provider readiness dependencies remain available, with invalid local references,
  self-dependencies, and cycles diagnosed; unsupported cross-scope topology is explicit rather than an indefinite wait.
- Dependency timeout and exhausted startup retries identify the waiting/failed service and leave surviving services
  reported, not silently torn down; runtime refresh follows
  `decision:running-environments/scope-environment/lookup-execution-policy`.
- Failed layout setup either safely tears down the newly created session or retains a clearly reported partial session;
  it is never reported as complete startup merely because some windows exist.

## Observe terminal health and output (product:tmux-observation)

A person can use terminal probes and choose persistence or terminal fidelity without losing trustworthy status.

Acceptance:

- URL, command, captured-log pattern, and process-uptime health checks remain available; stopped services with declared
  probes are not reported healthy, and undeclared probes remain distinguishable from failed observation.
- Log-pattern probes preserve regex meaning; uptime measures the service process rather than the persistent pane shell
  and resets when that process restarts.
- Declared literal and environment-relative ports appear when resolvable; reporting a port does not claim Winter bound
  it or proved it free. Scope-dependent port/probe limits are explicit.
- Persistent timestamped file logs and terminal-preserving pane output are distinct modes; rotation, retention, shutdown
  survival, missing timestamps, and follow approximations are disclosed under
  `decision:running-environments/provider-tmux/log-mode-support`.
- A memory logging choice is either implemented with stated behavior or refused; acceptance of an unimplemented mode
  cannot silently discard logs. Following several selected services is fair and stops without stopping those services.

## Stop and recover only owned sessions (product:tmux-owned-recovery)

An engineer can stop or recover their session without killing a neighbor's similarly named services.

Acceptance:

- Partial stop/restart terminates the selected service descendants while retaining unrelated services and pane shells;
  whole shutdown also removes the owned session.
- Shutdown attempts graceful termination then bounded escalation; surviving or inaccessible processes are failures or
  explicitly unverified cleanup, not a warning hidden behind confirmed success.
- Missing sessions are safe no-ops; unreadable declarations do not authorize matching sessions by environment suffix.
- Recovery establishes workspace-and-scope ownership before signaling; if ownership cannot be proved, it refuses and
  explains recovery options rather than broadening the target.
- Missing configuration, interrupted layout, changed prefix, and a dirty/default tmux server are diagnosable with
  non-destructive guidance; unsupported OS process-inspection behavior is explicit.
