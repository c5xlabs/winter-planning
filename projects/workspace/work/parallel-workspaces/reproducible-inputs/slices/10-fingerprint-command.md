---
name: fingerprint-command
depends-on:
  - git-port
code:
  - winter-workspace/crates/core/src/files.rs::OsFiles
---

# `ws fingerprint` prints the definition's digest

Adds `ws fingerprint [--json]`, the tree and dirty reads, and the scratch files of
[content-identity.md](../spec/content-identity.md). Delivers `product:workspace-content-identity` and
`engineering:fingerprint-observation`, under `decision:parallel-workspaces/reproducible-inputs/fingerprint-boundary`.

## Change

- The file port gains `ScratchFiles` on `OsFiles` and `MemoryFiles`: a copy that keeps bytes and modification time,
  and an empty path, each in a uniquely named directory under the snapshot's `TMPDIR`, else `/tmp`, removed on drop.
- The program port's `Call` gains `env`, set over the child environment for that call alone, which `tree` and `dirty`
  use for `GIT_INDEX_FILE` and `GIT_OPTIONAL_LOCKS`.
- The read git port gains `tree`, through a scratch copy of the real index or a scratch path seeded by `read-tree`, and
  `dirty`, through a status that takes no optional locks, each with its operation span.
- The command fingerprints the workspace root and each standalone sorted by name, digests the canonical preimage with
  SHA-256, and prints the digest or the shape A document.
- The library gains a SHA-256 dependency and `tempfile`, listed in the architecture context's dependency table.

## Acceptance

- Unit tests of the digest: a fixed preimage with an ASCII name, one with `é`, and one with a character outside the
  Basic Multilingual Plane each digest to the value Python's `json.dumps` and `hashlib.sha256` give.
- Component tests: two clones of one definition with different histories give one digest; a staged addition, a staged
  rename, a staged removal, and a same-size edit written in the index's own second each change it; an untracked file, a
  reordered standalone declaration, a change under `projects/`, and a new feature environment do not.
- Component tests: the real index's bytes and mtime, every ref, and every working file are unchanged after a run, with
  the remote unreachable; no scratch directory remains; `--json` carries `digest`, `workspace`, and `standalones` with
  `name`, `commit`, `tree`, and `dirty`.
- Component tests: a missing standalone checkout is `RepoMessage::MissingCheckout` with the reference's wording; a
  workspace root that is not a repository is its `RepoError`; each exits 1.
- `cargo dev gate slice` exits 0.
