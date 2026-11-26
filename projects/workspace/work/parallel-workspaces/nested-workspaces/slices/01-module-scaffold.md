---
name: module-scaffold
depends-on:
  - epic:parallel-workspaces/workspace-bootstrap
code:
  - winter-workspace/crates/core/src/termination.rs::RepoMessage
  - winter-workspace/crates/dev/src/architecture.rs::COMMAND_FILE_NAMES
---

# The nested workspace modules and error variants

Lands, in one change, the shared edits `context/architecture/changes.md` requires first of work built as parallel
changes: the module files present and empty, and the error variants the later slices render. These are the modules of
[child-state.md](../spec/child-state.md), [delegation.md](../spec/delegation.md), and
[teardown.md](../spec/teardown.md), Domain, under the `nested` module of [invocation.md](../spec/invocation.md), Domain,
which this slice declares and to which `child-environment` moves the scrub that
`epic:parallel-workspaces/workspace-bootstrap` landed in `bootstrap/checkout.rs`; with the `RepoMessage` variants
invocation.md and delegation.md list. Shaped by `engineering:typed-outcomes`.

## Change

- `lib.rs` declares `nested`; `crates/core/src/nested.rs` declares the `pub(crate)` submodules `state`, `delegate`, and
  `teardown`, each an empty file under `nested/`, so each later slice adds to its own file and re-exports from it on a
  `pub use` line of its own; the entries of later slices are re-exported by those slices, as the visibility paragraphs
  of invocation.md, child-state.md, and delegation.md, Domain, list. The `RepoMessage` variants are reached through
  `RepoMessage`, which `lib.rs` re-exports already.
- `context/architecture/structure.md` gains the rows of `nested` and its three submodules and what each may call;
  `changes.md` gains the rows its tables require for each.
- `nested::` joins `COMMAND_FILE_NAMES` in `crates/dev/src/architecture.rs`, its pinned test renamed for the new count:
  `env_create`, `env_destroy`, and the `ws status` collection call it, and no command file does.
- `RepoMessage` gains `OwnRootOnChain`, `RootOnChain`, `NoNestedConfig`, `NestedStatusFailed`, `NestedStatusNotJson`,
  `NestedStatusShape`, `NestedRootMismatch`, `NestedConfigUnreadable`, and `NestedNotStarted` of invocation.md, and
  `DelegationRefused` and `NestedCommandFailed` of delegation.md.
- Nothing constructs the new variants yet; no command changes.

## Acceptance

- Unit tests of the rendering: `OwnRootOnChain`, `RootOnChain`, `NoNestedConfig`, `NestedRootMismatch`, and
  `NestedNotStarted` read as invocation.md gives them, each naming its roots; `NestedRootMismatch` with no reported root
  still names the nested root; `NestedStatusShape` carries the field and what it held; `NestedCommandFailed` reads
  "nested `<command>` exited with code <returncode>".
- Unit tests of `DelegationRefused`: with `envs_fit` of 1 or more the text ends with the `envs = <n>` advice, and with
  none it ends with the advice to lower the nested workspace's `ports_per_env`, each naming the used and available ports
  and the outer `ports_per_env` to raise to, as delegation.md gives.
- A unit test of the command-file scan in `crates/dev`: a command file that names `nested::` is a violation naming the
  file, the line, and the token.
- `cargo dev gate slice` exits 0.

## Out of scope

- The `Action` variants of delegation.md and teardown.md land with the slices that emit them: `nested-init-step`,
  `teardown-preview`, and `child-retirement`.
