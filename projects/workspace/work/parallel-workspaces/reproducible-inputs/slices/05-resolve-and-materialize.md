---
name: resolve-and-materialize
depends-on:
  - git-port
  - lock-file
---

# Resolving a ref and materializing its lock entry

Fills `pins/resolve.rs` and `pins/materialize.rs` with the Resolution interface and the Materialize a pin flow of
[standalone-pins.md](../spec/standalone-pins.md), and `pins/outcome.rs` with the outcome types and the `RepoError`
rendering that `update-command` and `pull-advance` share: the resolution order of `product:standalone-reference-policy`,
and what the `ws init` of `epic:parallel-workspaces/workspace-bootstrap` calls to deliver
`product:parallel-workspaces/workspace-bootstrap/recorded-input-materialization`, under
`decision:parallel-workspaces/reproducible-inputs/resolution-and-locks`. Nothing calls `materialize` here.

## Change

- `resolve` tries the `origin` branch, then the tag, then the commit, answering the first match, and refuses a ref none
  of them match with `RepoMessage::UnresolvableRef`; it opens one `git rev-parse` span over its verifies.
- `resolve` and `materialize` take `Ports`; `materialize` also takes the workspace root and the loaded
  `StandaloneRepository`. It does nothing for a standalone whose `reference` is `None`; checks out a fresh entry without
  re-resolving, detached at the recorded commit for a tag or commit and onto the tracking branch for a branch; and
  re-resolves a missing or stale entry, refusing a dirty checkout with `RepoMessage::DirtyRePin`, then checks out and
  upserts the entry, in that order.
- A lock error is `PinError::Config`, a git failure or refusal `PinError::Repo`, and a git that could not be started
  `PinError::Internal`; the checkout's failure writes no entry.
- `pins/outcome.rs` holds `PinResult`, `PinOutcome`, and `rendered`, the `RepoError` as Python's `str()` renders it, as
  standalone-pins.md, Domain, gives them.
- `resolve`, `materialize`, and the outcome names are re-exported by name from `lib.rs` for their tests.

## Acceptance

- Component tests of `resolve`: a name that is both a remote branch and a tag resolves as the branch; a lightweight tag
  as its commit; an annotated tag as its tag object's id; a short SHA as the full commit; an unknown name is the
  unresolvable-ref error.
- Component tests of `materialize`: a fresh tag entry checks out its commit after the tag moved on the remote; a fresh
  branch entry lands on the last-fetched `origin` tip and answers the recorded commit with the lock unchanged; a stale
  entry with an untracked file is refused with the lock unchanged; a stale clean entry is re-resolved and rewritten; a
  missing entry is resolved and written beside another standalone's entry, which is kept; `ref = ""` is pinned and
  refused as unresolvable.
- A component test: a malformed lock is `PinError::Config`, with the checkout's HEAD and the lock's bytes unchanged.
- A component test with tracing on: a ref resolved as a commit records one `git rev-parse` span.
- Component tests: a fresh commit entry whose commit is absent from the object store fails with the lock's bytes
  unchanged; a stale entry whose checkout git refuses, an untracked file in the way of the new commit's tracked file, is
  `PinError::Repo` with the lock's bytes and the checkout's HEAD unchanged.
- Unit tests of `rendered`: a `RepoError` with every field gives the message and the `$`, `cwd:`, `exit`, and `stderr:`
  lines, each indented by two spaces, joined by newlines with none after the last; an exit of -9 adds
  `(killed by signal 9)`; absent fields give no line; stderr is trimmed.
- `cargo dev gate slice` exits 0.
