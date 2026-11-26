---
name: status-nested-member
depends-on:
  - status-read
  - epic:parallel-workspaces/coordinated-git
---

# The `nested` member of `ws status`

Adds the `nested` member of a worktree row of [child-state.md](../spec/child-state.md), The `nested` member of
`ws status`: `WorktreeRow` gains the field that [status.md](../../coordinated-git/spec/status.md), Domain, reserves for
it; `collect`, which `epic:parallel-workspaces/coordinated-git` builds, reads `Nesting::state` of `status-read` for each
existing worktree of a nested project as status probes of [fan-out.md](../../coordinated-git/spec/fan-out.md); the text
adds one line per member; and the exit counts a dirty or unreadable child as a dirty worktree. Delivers the `ws status`
bullets of `product:nested-work-inspection`.

## Change

- `WorktreeRow` gains `nested: Option<NestedMember>`; the document writes it only on the rows child-state.md names.
- `collect` runs the nested reads after the env and worktree probes, at most `STATUS_PROBES` at once, each result into
  its own row; a `RepoError` becomes the member's `error`, never the collection error.
- The text writes the `(nested)` lines after each env's table, and `exit_code` counts a member with `dirty` true or an
  error as a dirty worktree.
- `crates/core/topics/ws/status.md` describes the member.

## Acceptance

- A component test with real git and a `winter` stand-in writing a child document of two envs, one dirty: the row of the
  nested worktree holds `{"env_count": 2, "dirty": true, "unpushed": false, "error": null}`, no other row has the
  member, the document validates against the copied v1 schema, and the text line is
  `"  <repo> (nested): 2 envs, dirty"`.
- A component test: a child with no `.winter/config.toml` gives `env_count` null and the rendered `NoNestedConfig` as
  `error`, the text line reads "unreadable — <first line>", the other rows and environments are all present, and the
  command exits 1, not 2.
- A component test: a child whose document gives a branch `ahead` with no upstream gives `unpushed` true; with nothing
  else, the exit code is 0.
- A component test: a nested project whose worktree is absent has no member and starts no `winter` call.
- A unit test over a recording program port: with ten nested worktrees, never more than 8 nested reads in flight, and
  the rows keep configuration order whatever the completion order.
- `cargo dev gate slice` exits 0.
