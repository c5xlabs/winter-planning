---
name: child-state-parse
depends-on:
  - module-scaffold
---

# Reading a child's status document

Adds `NestedState`, `NestedEnv`, `NestedMember`, `parse`, `reported_root`, and `ShapeFault` of
[child-state.md](../spec/child-state.md), Domain, Shape rules, and Derivation: the pure read of a child's v1
`ws status --json` document into the environments, dirty repositories, and unpushed entries the inspection and the loss
audit share. Delivers the derivation half of `product:nested-work-inspection` and of `engineering:nested-loss-evidence`,
under `decision:parallel-workspaces/nested-workspaces/child-compatibility`, whose unknown-not-zero reading is a
registered deviation.

## Change

- `crates/core/src/nested/state.rs` holds the types and the two functions; `reported_root` checks the object and
  `schema_version` 1 first, and `parse` checks every field the Shape rules name, ignoring any other.
- `parse` takes `present`, the standalones the child declares that exist on disk, and adds a `(state unknown)` checkout
  for each one the document leaves out; it reads no file itself.
- `NestedState`, `NestedEnv`, `NestedMember`, `ShapeFault`, and `reported_root` are re-exported by name from `lib.rs`,
  and `parse` as `parse_nested_state`, for its tests ([child-state.md](../spec/child-state.md), Domain); no command
  calls them yet.

## Acceptance

- Unit tests of the shape rules: a `schema_version` of 2, of `"1"`, or absent is the "has schema_version" fault; a count
  that is negative, a float, or a boolean is a `ShapeFault` naming its field path; `tracking_ref_present` is not read
  when `upstream` is null; an extra field at every level is ignored.
- Unit tests of an env worktree: `tracking_ahead` above 0 is "unpushed commits"; `ahead` above 0 with a null upstream,
  and with an upstream whose tracking ref is absent, is "unpushed branch"; `ahead` above 0 with its tracking ref present
  and `tracking_ahead` 0 has no reason; a `nested` member with `unpushed` true adds "unpushed work in its nested
  workspace", and one with `dirty` true or a string `error` makes its repository dirty.
- Unit tests of a checkout: each reason appears in the spec's order under its `projects/<repo>` or `standalone <repo>`
  label; an absent `stashes` or `local_only_commits` reads as "stashes unknown" or "local-only commits unknown", never
  as none.
- A unit test: a standalone in `present` with no row in `standalones` is "standalone <repo> (state unknown)"; one not in
  `present` with no row adds nothing.
- Unit tests: `env_count`, `dirty`, and `unpushed` answer from the envs and checkouts as the Domain gives, with
  `unpushed` entries `<env>/<entry>` before the checkouts.
- `cargo dev gate slice` exits 0.
