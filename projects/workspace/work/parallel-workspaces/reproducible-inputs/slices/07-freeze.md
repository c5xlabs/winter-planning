---
name: freeze
depends-on:
  - update-command
---

# `ws update --freeze` pins checkouts as they are

Adds `--freeze` and `--force` to `ws update`, running the Freeze flow of
[standalone-pins.md](../spec/standalone-pins.md), and builds the `Set a standalone ref` writer of
[configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md)
beside the delivered removal, through its write integrity. Delivers `product:freeze-standalone-inputs`.

## Change

- `--force` without `--freeze`, and `--freeze` with `--autostash`, are the usage errors the command surface gives.
- Selection takes any standalone a pattern matches, pinned or not, with the update's glob rule, and refuses an unknown
  literal name as `RepoMessage::NoStandaloneNamed`.
- A lock error after the `ref` is written ends the run as `PinError::Config`, the config file already changed, with
  no `pull_completed`, as the reference does.
- Each selected standalone reports `already_pinned`, `refused` with the reference's reasons, or `pinned`, after `ref`
  is inserted after the last key of the first block whose effective name matches, in the file that declares it, and
  the commit entry is upserted.
- Nothing fetches and nothing checks out.

## Acceptance

- Component tests: an unpinned standalone declared in `config.toml` gains `ref = "<sha>"` after its last key with the
  file's other bytes unchanged, and the lock gains a `commit` entry; one declared only in `config.local.toml` is written
  there; one declared by `url` alone is matched by its derived name; a pinned one reports `already_pinned` with both
  files unchanged.
- Component tests: a dirty checkout is `refused` without `--force` and `pinned` with it; a missing checkout and a
  directory that is not a repository are `refused` by name; each refusal exits 1.
- Component tests: a commit not present on the remote is pinned; with the remote unreachable, every freeze succeeds and
  the checkout's HEAD and refs are unchanged.
- A component test: with a malformed `.winter/config.lock`, a freeze exits 1 with the configuration error, the
  standalone's `ref` written to `config.toml`, and no `pull_completed`.
- Component tests of the `pinned`, `already_pinned`, and `refused` rows in text and `--json`, and of the two usage
  errors, exit 2.
- `cargo dev gate slice` exits 0.
