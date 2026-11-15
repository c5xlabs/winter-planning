---
name: prove-dispatch-boundaries
depends-on:
  - wire-launcher-dispatch
---

# Prove process preservation and overlapping legacy launches with fixtures

Complete cross-process evidence for `engineering:fixture-only-dispatch` and `engineering:legacy-adapter-boundary`,
covering `product:preserve-invocation`, `product:invocation-aware-help`, `product:legacy-workspace-dispatch`,
`product:legacy-init`, and `product:singleton-cutover`. Exercise the handoff and lease contracts in
[Interfaces](../spec.md#interfaces) and [Flows](../spec.md#flows) under `decision:launcher-shim/component-boundary`,
`decision:launcher-shim/venv-ownership`, and `decision:launcher-shim/manual-cutover`.

## Change

- Add fixture product/Python entrypoints and stub uv modes to the foundation's bounded test harness; never use the
  installed winter, real Python Workspace, live Fleet services, or operator selections.
- Model handoff version 1 in modern stand-ins: read a nonempty invocation prefix once, remove it before launching
  children, and use it only for help/examples/nested help/usage errors, not identity/version.
- Exercise real CLI replacement, streams, status, signals, and native argv/environment with decoy executables and
  complete isolated ancestor trees.
- Coordinate overlapping warm readers, competing cold starts, and A/B source switches at observable fixture boundaries,
  including the exclusive-to-shared revalidation window.
- Prove setup cancellation and lease lifetime/release through actual replacement, failure, and signal termination.

## Acceptance

- Modern and legacy fixture PIDs equal the spawned launcher PID; cwd, native args/env, stdin/stdout/stderr bytes,
  nonzero exits, and SIGINT/SIGTERM termination match their route contracts without a surviving component.
- Shim help, nested help, examples, and usage errors show the invoked role prefix; direct/empty-prefix invocation uses
  the component name, identity/version stay stable, and child fixtures never inherit `WINTER_INVOKED_AS`.
- Concurrent warm Python readers coexist while stale synchronization fails `Busy` without mutation. Cold-launch
  contention stays bounded; only an exclusive owner runs uv or changes the stamp/environment.
- A/B/A launches with older source mtimes synchronize on each identity switch; relocated/symlink-alias venvs share one
  lease. An intervening source switch before shared reacquisition yields `Busy`, never wrong-source execution.
- The shared lease survives exec and blocks another setup until normal exit or signal; failed exec releases it.
  Synchronizer children cannot retain the lease descriptor.
- Cancellation during stub sync propagates interruption, kills/reaps setup children, leaves no valid stamp, and forces
  retry synchronization. Failed sync/publication cannot launch Python or reuse partial setup as warm.
- Bootstrap absent/existing-config and source-override failures preserve operator content, never consult ancestor
  eligibility, and never grant ordinary legacy permission.
- Missing/non-executable/invalid-format targets remain launcher failures distinct from started-component failures. PATH
  decoys stay untouched; allowed fixture uv effects are distinguished from forbidden Rust/install/config writes.
- Unit, component, and service tiers pass in separate production/development configurations. Timeout/failure cleanup
  leaves no owned fixture process, and no verification reads real root-to-cwd selections.

## Out of scope

- Manual installation/rollback, real Workspace adoption, production component implementation, release publishing, and
  legacy retirement are not prerequisites for this fixture evidence.
