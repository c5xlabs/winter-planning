# Workspace configuration engineering requirements

For `epic:parallel-workspaces/workspace-configuration`.

## What already holds

- `decision:configuration-transition` fixes the four files and their formats; `decision:functional-parity` and
  `decision:visible-gap-policy` fix what may differ from the reference tool.
- `epic:parallel-workspaces/component-runtime` supplies the endings and their exit codes, the file port, and the rule
  that the workspace is located and its configuration loaded only when a command first needs it.
- `epic:rust-adoption/parity-validation` built `ws index` and `repo remove` on a reader of two keys and on the
  replacement of one file; this epic puts the whole model under both.
- The write path of [write-path.md](./spec/write-path.md) exists in `crates/core/src/files.rs` of the code repository.
  Its tests already hold two of its rules: `a_link_to_a_file_that_does_not_exist_is_not_written_through_and_stays` in
  `files.rs`, and `the_command_waits_while_another_handle_holds_the_lock_of_the_directory` in
  `crates/core/tests/repo_remove.rs`.
- A recorded scenario compares the exit code, the parsed JSON, and the tree, and neither stderr nor text output
  (`context/verification/scenarios.md` of the code repository). The kind of a refusal and its message therefore rest on
  unit tests, and the number rendering of `[env]` on a unit test table until a command prints an environment.
- The scenario runner defers an interruption and two overlapping invocations, and a fixture holds no hard link, no
  foreign owner, no mount, and no permission bits beyond the execute bit. The write path therefore rests on the unit
  tests of `files.rs`, which run in temporary directories, and on component tests of `repo remove`.
- No public function of the library returns a `WorkspaceConfig` until a command reads one. A test that asserts its
  fields is a unit test of the `core` crate over the in-memory file port; a component test reaches the load only through
  the ending of a command.
- A unit test of the code repository takes no file input (`context/standards/testing.md` there); the unit tests of
  `files.rs` are the exception that touches the file system.

## Consistent invocation snapshot (engineering:configuration-snapshot)

One invocation locates its workspace once, reads each configuration file once, and uses that one validated value for
target resolution and for effects; no command reads a key from a file a second time. A writer does not edit the
snapshot: it reads the file again under the write lock, so an edit made by another process since the load is kept.
Serves `product:identify-workspace` and `product:configuration-validation`.

Acceptance:

- A lookup of an env index reads each of the two configuration files exactly once through the read port, so a file
  replaced after that read cannot change the run.
- A removal whose file gained a repository entry from another writer between the load and the lock removes its own entry
  and keeps the other writer's.
- A removal started while another handle holds the lock of the directory writes nothing until the lock is released, and
  then completes.

## Side-effect-free interpretation (engineering:inert-configuration-read)

Loading the configuration starts no process, writes no file, creates no directory, and takes no lock; a failure names
the file or the key at fault and never carries the value of an environment variable entry. Serves
`product:effective-configuration`.

Acceptance:

- Loading a configuration whose environment table holds command entries leaves the tree of the workspace as it was and
  starts no process.

## Preserve durable edits (engineering:configuration-write-integrity)

Supported configuration writers preserve unmanaged content and coordinate across processes; interruption cannot turn a
valid previous declaration into an accepted partial declaration. Conflict and recovery policy follows
`decision:parallel-workspaces/workspace-configuration/write-conflicts`; the bytes outside an edit are kept under
`decision:parallel-workspaces/workspace-configuration/write-byte-preservation`. Serves
`product:configuration-state-ownership`.

## Every refusal is typed (engineering:typed-configuration-failures)

A configuration the binary refuses ends as a configuration error, exit 1, whose kind carries the file or the key at
fault and the values its message names; no input that reaches the loader ends as an internal failure. This holds for
every input on which the reference tool ends with an unhandled error, under
`decision:parallel-workspaces/workspace-configuration/unhandled-reference-failures`. Serves
`product:configuration-validation`.
