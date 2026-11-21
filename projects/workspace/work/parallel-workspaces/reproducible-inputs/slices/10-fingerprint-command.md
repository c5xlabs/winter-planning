---
name: fingerprint-command
depends-on:
  - bounded-fetch
code:
  - winter-workspace/crates/core/src/files.rs::OsFiles
  - winter-workspace/crates/core/src/context.rs::Context
---

# `ws fingerprint` prints the definition's digest

Adds `ws fingerprint [--json]`, the tree and dirty reads, and the scratch files of
[content-identity.md](../spec/content-identity.md), its document shape A of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md) and its errors
the `Termination::Repo` of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md).
Delivers `product:workspace-content-identity` and `engineering:fingerprint-observation`, under
`decision:parallel-workspaces/reproducible-inputs/fingerprint-boundary`. Follows `bounded-fetch`, which also changes
`Call` and `git.rs`.

## Change

- The file port gains `ScratchFiles`, implemented on `OsFiles` alone: a copy that keeps bytes and modification time, and
  an empty path, each in a uniquely named directory under the `temp` directory the caller passes, removed on drop, named
  as `files.rs` names its own temporary files. `Context` gains `scratch`, which `run_with` fills with its `OsFiles`, and
  `ScratchFiles` joins `COMMAND_FILE_NAMES` beside `ReadFiles`, its pinned test renamed for the new count.
- The program port's `Call` gains `env`, set over the child environment for that call alone, which `tree` and `dirty`
  use for `GIT_INDEX_FILE` and `GIT_OPTIONAL_LOCKS`; every existing call passes an empty slice.
- `git.rs` gains `tree`, through a scratch copy of the real index or a scratch path seeded by `read-tree`, and `dirty`,
  through a status that takes no optional locks, each with its operation span.
- The command file calls `fingerprint::compute` with `cx.files`, `cx.scratch`, `cx.programs`, `cx.tracer`, `TMPDIR` of
  `cx.env`, else `/tmp`, and `cx.cwd`, with the signature content-identity.md gives. It fingerprints the workspace root
  and each standalone sorted by name, refusing a checkout that `ReadFiles::is_dir` denies as
  `RepoMessage::MissingCheckout` with the path as its `cwd`, ends a failure as the `GitFailure` content-identity.md
  gives, digests the canonical preimage with SHA-256, and the command prints the digest or the shape A document.
- The library gains a SHA-256 dependency, listed in the architecture context's dependency table.

## Acceptance

- Unit tests of the digest: a fixed preimage with an ASCII name, one with `é`, and one with a character outside the
  Basic Multilingual Plane each digest to the value Python's `json.dumps` and `hashlib.sha256` give.
- Unit tests of `ScratchFiles` inside a `TempTree`: a copy keeps the source's bytes and modification time; the directory
  is under the given `temp` and is gone after the handle drops.
- Component tests: two clones of one definition with different histories give one digest; a staged addition, a staged
  rename, a staged removal, and a same-size edit written in the index's own second each change it; an untracked file, a
  reordered standalone declaration, a change under `projects/`, and a new feature environment do not.
- Component tests: the real index's bytes and mtime, every ref, and every working file are unchanged after a run, with
  the remote unreachable; no scratch directory remains under the isolated tree's `tmp/`; `--json` carries `digest`,
  `workspace`, and `standalones` with `name`, `commit`, `tree`, and `dirty`.
- A component test with a stub `git` first on the `PATH` that writes its environment to `calls/`: the `read-tree` call
  of a repository without an index, `add -u`, and `write-tree` see `GIT_INDEX_FILE` under the isolated tree's `tmp/`;
  `rev-parse --git-path index`, `rev-parse HEAD`, and `status` do not.
- Component tests: a missing standalone checkout is `RepoMessage::MissingCheckout` with the reference's wording; a
  workspace root that is not a repository is its `RepoError`; each exits 1.
- `cargo dev gate slice` exits 0.
