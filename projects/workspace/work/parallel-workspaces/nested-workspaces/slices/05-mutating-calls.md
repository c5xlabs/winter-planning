---
name: mutating-calls
depends-on:
  - status-read
  - epic:parallel-workspaces/workspace-bootstrap
---

# Running a mutating command in a verified child

Adds `Nesting::run` and `Nesting::binds_service` of [invocation.md](../spec/invocation.md), Mutating calls, with the
service-binding read of [teardown.md](../spec/teardown.md), Phases, `binds_service`: `winter <args>` in a child root
only after `ensure_verified` of `status-read` passes, its arguments passed as given and its merged output delivered line
by line as reconciliation.md, Commands, streams a `cmd`, which `epic:parallel-workspaces/workspace-bootstrap` builds.
Delivers the mutating half of `engineering:child-authority-confinement` and the no-shell rule of
`engineering:child-process-isolation`.

## Change

- `run` verifies, then starts `Program::Winter` in the root with stdin inherited, stdout and stderr on one pipe, and no
  timeout; it answers the returncode, minus the signal number for a signal, or `NestedNotStarted`.
- `binds_service` reads the child's two configuration files as `status-read` does and answers whether `capabilities`
  binds a non-empty `service`, as a string or within an array.
- `run` and `binds_service` are reached through `Nesting`, re-exported from `lib.rs` by `status-read`, for its tests
  ([invocation.md](../spec/invocation.md), Domain); no command calls them yet.

## Acceptance

- Component tests with a `winter` stand-in: `run` of a verified root starts the stand-in once with the arguments given,
  an argument holding a space and a `;` arriving as one argument; each output line arrives in order, stderr lines
  included; a non-zero exit answers its code.
- A component test: `run` of a root whose status read reports another root is `NestedRootMismatch`, and the stand-in
  records the status read alone.
- A component test: `run` of a root already verified in this `Nesting` starts no second status read.
- Unit tests over `MemoryFiles` of `binds_service`: `service = "tmux"` and `service = ["docker"]` are true; an empty
  string, an empty array, no `capabilities`, and no files are false; a local overlay binding over a committed file
  without one is true; a file that is not TOML is `NestedConfigUnreadable`.
- `cargo dev gate slice` exits 0.
