---
name: nested-init-step
depends-on:
  - mutating-calls
  - local-key-upsert
  - delegation-values
  - epic:parallel-workspaces/environment-lifecycle
---

# The nested step of `ws init <env>`

Adds `delegate::reconcile` of [delegation.md](../spec/delegation.md), The nested step and Moves, as step 6 of one
worktree in [init.md](../../environment-lifecycle/spec/init.md), One worktree, which
`epic:parallel-workspaces/environment-lifecycle` builds: for an env worktree of a project with `nested` declared whose
`cmd` succeeded, verify the child, write the delegated and inherited values into its local overlay through `update` of
[write-path.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/write-path.md) under the child's
`.winter/` lock, then run the child's bare `winter ws init`. Delivers the initialization bullets of
`product:verified-child-workspace`, `product:nested-runtime-allocation`, and `product:nested-local-inheritance`, and
`engineering:delegation-write-integrity`, under `decision:parallel-workspaces/nested-workspaces/delegation-policy`,
whose named move is a registered deviation.

## Change

- `crates/core/src/nested/delegate.rs` gains `reconcile`: `ensure_verified`, the outer local and child committed reads,
  then one `update` whose edit refuses a footprint above the slice, reports a move, and applies `local-key-upsert`, then
  `run` of `ws init` between `CmdStarted` and `CmdCompleted`.
- `bootstrap.rs`'s `Action` gains `NestedAllocationMoved` and `NestedOverlayWritten`, with their text.
- `reconcile_env` holds one `Nesting` for the invocation and `worktree::run` calls `reconcile` as step 6, skipped when
  `cmd` failed; `ws init --all` reaches it through `reconcile_env` for each environment.
- `crates/core/topics/ws/init.md` says what `ws init <env>` does for a nested project.

## Acceptance

- Component tests with real git and a `winter` stand-in that answers a verified status and records its calls:
  `ws init alpha` with one nested project writes `base_port`, `service_prefix`, and the parent's `[git]` into the
  child's `config.local.toml`, reports `nested_overlay_written` naming them, then runs `winter ws init` in the child,
  whose lines arrive as `CmdOutputLine`; `ws init --all` does so for each environment.
- A component test: two nested projects receive disjoint slices of the env band, neither holding the parent's slice 0,
  and different service prefixes.
- Component tests: a child whose footprint exceeds its slice, and a slice of size 0, fail the repository with
  `DelegationRefused` before any write and before the child's `ws init`; a child overlay that is not TOML fails with
  `NestedConfigUnreadable` and its bytes unchanged.
- A component test: a stand-in reporting another root fails the repository with `NestedRootMismatch`, writes nothing,
  and starts no `ws init`.
- A component test: adding a second nested project ahead of the first moves it; `nested_allocation_moved` naming
  `base_port <old> -> <new>` is emitted before the overlay changes, and the write applies.
- Component tests: a child overlay holding its own keys and comments keeps them; a key inherited by an earlier init and
  no longer named stays as it was; `inherit_local = []` copies nothing and keeps what the child holds.
- A component test: an `inherit_local` naming `base_port` and `service_prefix`, both set in the parent's local file,
  writes the delegated values for them, not the parent's.
- A component test: with `alpha` recorded in the env index at an index other than its hash suggestion, the delegated
  `base_port` is the slice of the band `port_base` gives for the recorded index.
- A component test: a child `ws init` exiting 1 fails the repository with `NestedCommandFailed`, and the other
  repositories of the environment go on.
- A component test: a failing `cmd` of the nested project starts no `winter` call.
- A component test: bare `ws init` of a workspace with a nested project starts no `winter` call, and the loaded
  configuration holds the nested project as no extension.
- A component test: eight threads, each adding a key of its own to the child's overlay through `update` while the step
  runs, leave all eight keys and the delegated values in the file.
- `cargo dev gate slice` exits 0.
