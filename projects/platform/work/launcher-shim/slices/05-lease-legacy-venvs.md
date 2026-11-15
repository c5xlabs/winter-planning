---
name: lease-legacy-venvs
depends-on:
  - production-process-contract
---

# Lease physical venv destinations and track source-aware freshness

Build the lease and stamp mechanisms from legacy [Flows](../spec.md#flows) and [Data](../spec.md#data), serving
`product:legacy-workspace-dispatch`, `engineering:legacy-adapter-boundary`, and `engineering:fixture-only-dispatch`.
`decision:launcher-shim/venv-ownership` requires bounded contention and exclusive ownership before any venv/stamp
mutation.

## Change

- Resolve nonempty `UV_PROJECT_ENVIRONMENT` relative to selected-cli, or use selected-cli's `.venv`; resolve symlinks in
  the longest existing ancestor and append the missing suffix.
- Open the stable sibling `.winter-launch.lock` without truncating/unlinking it, create missing parent directories, and
  allocate close-on-exec lease descriptors above fd 2.
- Implement nonblocking shared/exclusive leases and explicit release/reacquire transitions; contention is `Busy`, other
  I/O failures are `Setup`.
- Read source-aware freshness from entrypoint executability, exact stamp bytes/source identity, and dependency-input
  mtimes. Treat stamp I/O failures as setup errors, not warmth.
- Provide exclusive-only invalidation and atomic same-directory stamp publication with temporary-file cleanup, plus the
  freshness rechecks needed after each lease transition. No synchronizer or public route calls this mechanism yet.

## Acceptance

- Default/relocated/relative/symlink-alias fixtures resolve to the physical destination and one stable lease identity,
  including destinations whose suffix does not yet exist.
- Concurrent warm readers coexist; exclusive acquisition against any reader/writer fails promptly without changing venv
  or stamp. The sibling lock survives venv replacement and is never unlinked.
- Missing/non-executable entrypoints, absent/malformed/old-format stamps, different native source identities, and each
  newer dependency input are stale; warm sources with unchanged inputs remain warm.
- A/B/A source identities require refresh even when their dependency mtimes are older than the previous stamp.
- Publication uses the exact lowercase-hex source format and completion mtime; failed publication leaves no valid marker
  or unpublished temporary file. Failed invalidation prevents synchronization and leaves the venv untouched.
- Descriptor fixtures retain close-on-exec during preparation and preserve closed standard streams. Lease rechecks
  detect an intervening source/stamp/input change rather than treating flock conversion as atomic.

## Out of scope

- `prepare-legacy-launch` owns when these operations surround uv and bootstrap. `prove-dispatch-boundaries` covers
  overlapping real launcher invocations and shared ownership across Python replacement.
