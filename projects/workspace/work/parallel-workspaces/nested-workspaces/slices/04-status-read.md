---
name: status-read
depends-on:
  - child-state-parse
  - child-environment
---

# Verifying a child and reading its state

Adds `Nesting`, `Nesting::new`, `state`, and `ensure_verified` of [invocation.md](../spec/invocation.md), Preflight and
Status read and verification: the chain and marker guards every nested call runs before starting anything, then
`winter ws status --json` in the child, accepted on exit 0 or 1 only when its `root_path` resolves to the child root,
remembered as verified for the rest of the invocation. `state` then lists the child's declared standalones present on
disk, as [child-state.md](../spec/child-state.md), Derivation, unseen standalone, gives, and answers `parse` of
`child-state-parse`. Delivers the verification and refusal bullets of `product:verified-child-workspace`,
`engineering:child-authority-confinement` for the read, and the unknown-state bullets of
`product:nested-work-inspection`, under `decision:parallel-workspaces/nested-workspaces/child-compatibility`.

## Change

- `crates/core/src/nested.rs` gains `Nesting`, shared by every nested call of one invocation and across fan-out threads,
  with its `verified` set behind a mutex.
- Preflight runs in the spec's order: own root on the chain, then the target root on the chain, then a missing
  `.winter/config.toml`, each a `RepoError` with nothing started.
- The status read captures stdout and stderr whole with no timeout and maps every outcome the spec lists to its
  `RepoMessage`.
- `present` reads the child's `.winter/config.toml` and `.winter/config.local.toml` merged as configuration-model.md,
  Layering, gives, and checks each standalone's location of location-listing.md under the child root; either file not
  valid TOML is `NestedConfigUnreadable`.
- No command calls it yet.

## Acceptance

- Component tests with a `winter` stand-in in the test tree's `bin/` writing a fixed document: exit 0 and exit 1 both
  verify; exit 2 and a signal are `NestedStatusFailed` with the child's stderr; stdout not JSON is
  `NestedStatusNotJson`; `schema_version` 2 is `NestedStatusShape`; a `root_path` naming another directory, and none,
  are `NestedRootMismatch`; no `winter` on `PATH` is `NestedNotStarted`.
- Component tests: with the own root on `WINTER_NESTED_CHAIN` the call is `OwnRootOnChain`, with the child root on it
  `RootOnChain`, and with no `.winter/config.toml` in the child `NoNestedConfig`; in each the stand-in records no start.
- A component test: two `ensure_verified` calls for one root read once; `state` reads again each time; a fresh `Nesting`
  reads again.
- A component test: a child declaring a standalone that exists on disk and that its document leaves out answers a
  `(state unknown)` checkout; one declared only in its local overlay counts; one absent from disk adds nothing; a child
  `config.local.toml` that is not TOML is `NestedConfigUnreadable`.
- A component test: the stand-in sees `WINTER_NESTED_CHAIN` ending with the outer root and none of the removed names.
- `cargo dev gate slice` exits 0.
