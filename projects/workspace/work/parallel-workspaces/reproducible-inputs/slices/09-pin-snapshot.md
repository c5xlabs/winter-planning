---
name: pin-snapshot
depends-on:
  - git-port
  - lock-file
  - standalone-refs
---

# The pin snapshot of each pinned standalone

Builds the Pin snapshot flow of [standalone-pins.md](../spec/standalone-pins.md): the inspectable-resolution bullet of
`product:standalone-reference-policy`. The workspace status of `epic:parallel-workspaces/coordinated-git` is its
reader; nothing calls it here.

## Change

- One `PinSnapshot` per standalone that declares a `ref`, in merged order, from one read of the lock and a HEAD read
  per checkout.
- No pinned standalone reads no lock; an unreadable lock is logged and read empty; an unreadable HEAD leaves
  `head_commit` empty and `head_drift` false.
- `snapshot` is re-exported by name from `lib.rs` for its component tests.

## Acceptance

- Component tests: an entry whose `ref` differs from the declaration sets `config_ref_drift`; a checkout moved off the
  recorded commit sets `head_drift`; an annotated-tag entry sets `head_drift` at its own tag; a standalone without an
  entry has no `kind` and no drift.
- Component tests: a malformed lock yields snapshots without entries and a warning record; a missing checkout yields
  no `head_commit`; with no pinned standalone and a malformed lock, no warning is written.
- `cargo dev gate slice` exits 0.
