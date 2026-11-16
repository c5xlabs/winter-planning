# Write path

How the binary replaces one of its four workspace files: the port, the lock, the replacement, and every way it fails.
What each writer puts in a configuration file is [configuration-model.md](./configuration-model.md); the lock and state
file contents are [lock-and-state.md](./lock-and-state.md); the port's place in the crate is
[repository-layout.md](../../component-runtime/spec/repository-layout.md).

## System

The reference tool writes each file in place, with no lock and no temporary file
(`core/internal/local_filesystem.py:59-61`). The binary replaces the file under a lock, which
`decision:visible-gap-policy` allows as hardening and
`decision:parallel-workspaces/workspace-configuration/write-conflicts` fixes; it does so in every case and never falls
back to a write in place (`decision:parallel-workspaces/workspace-configuration/replacement-edges`). The port and its
operating-system implementation exist in `crates/core/src/files.rs`; this epic changes neither and adds tests to them.

## Interfaces

```rust
// engineering:configuration-write-integrity
pub trait WriteFiles {
    // `edit` receives the current bytes, None when no such file exists, and returns the new bytes or None for
    // no change. Ok(true) when the file was replaced, Ok(false) when the edit asked for no change.
    fn update(&self, path: &Path, edit: &mut Edit<'_>) -> io::Result<bool>;
}
```

A domain writer maps an `Err` to `ConfigError::FileWrite { path, cause }`
(`decision:parallel-workspaces/workspace-configuration/write-failure-kind`); a failure of the edit itself is the
writer's own error. Unit tests use the in-memory implementation of the same trait.

## Flows

```text
# decision:parallel-workspaces/workspace-configuration/write-conflicts          applies to all four files
update(path, edit):
    create the parent directory of path, and its ancestors, when absent
    lock:     that directory, `.winter/`, is opened for reading and an exclusive advisory lock is taken on the open
              handle; no lock file is created; the lock ends when the handle closes or the process dies; one lock
              serialises writers of all four files
              a writer that cannot take the lock waits, with no limit and no message; a lock call that a signal
              interrupts is made again
    resolve:  a path that is a symbolic link is resolved; the file the link points to is the target
              and the link itself is left in place, also when the target lies outside the workspace
    read:     the current bytes of the target, under the lock; None when nothing exists at the path, or when the
              path is a symbolic link to a file that does not exist
    edit:     returns None -> nothing is written; the lock is released
    write:    a new file `.<name>.<pid>.<n>.tmp` beside the target, created exclusively, given the permission bits
              of the target, written, flushed to the device, then renamed over the target
              a target that did not exist is created with the mode a plain create gives it
    failure:  the temporary file is removed; the target is as it was
readers:      take no lock; a reader sees the old file or the new file, never a partial one
```

| Failure                                                                           | Result                    | Ending                           | Exit |
| --------------------------------------------------------------------------------- | ------------------------- | -------------------------------- | ---- |
| The directory cannot be created, or cannot be opened for reading                  | `Err`, the system's error | Configuration error, `FileWrite` | 1    |
| The lock call fails                                                               | `Err`, the system's error | Configuration error, `FileWrite` | 1    |
| The path cannot be resolved, or the target cannot be read                         | `Err`, the system's error | Configuration error, `FileWrite` | 1    |
| The edit returns new bytes and the target exists and cannot be opened for writing | `Err`, the system's error | Configuration error, `FileWrite` | 1    |
| The edit returns new bytes and the path is a symbolic link to a missing file      | `Err`, not found          | Configuration error, `FileWrite` | 1    |
| The temporary file cannot be created, written, or renamed                         | `Err`, the system's error | Configuration error, `FileWrite` | 1    |
| The process is killed between create and rename                                   | the temporary file stays  | none; the target is unchanged    | —    |

An edit that returns `None` is never refused for the target: a file that cannot be opened for writing, and a link to a
missing file, are then left as they are and the call answers that nothing was replaced.

A temporary file left by a killed writer is never read and never removed
(`decision:parallel-workspaces/workspace-configuration/abandoned-temporary-files`); its name starts with a dot and ends
in `.tmp`, so no reader of the four files meets it.

## Data

Where a replacement can be told from the reference tool's write in place, with no concurrent writer. The first nine rows
are the registered deviations of `decision:parallel-workspaces/workspace-configuration/replacement-edges`, listed in
[command-surface.md](../../component-runtime/spec/command-surface.md); each is stated as `crates/core/src/files.rs`
behaves. The last is no difference.

| Case                                                    | Reference tool                 | Binary                                                             |
| ------------------------------------------------------- | ------------------------------ | ------------------------------------------------------------------ |
| The target has a second hard link                       | Both names see the new content | The other name keeps the old content                               |
| The target is owned by another user or group            | Owner and group kept           | The new file has the owner and group of a file the process creates |
| The target is writable, its directory takes no new file | Written                        | Refused: the temporary file cannot be created                      |
| The path is a symbolic link to a missing file           | The missing file is created    | Refused, not found; the link stays and nothing is created          |
| The lock call on the directory fails                    | Written; no lock is taken      | Refused with the system's error                                    |
| Another writer holds the lock                           | Written at once                | Waits until the lock is released, with no limit, then writes       |
| The directory cannot be opened for reading              | Written                        | Refused: the handle the lock is taken on cannot be opened          |
| The target is a mount point                             | Written in place               | Refused when the system refuses the rename over it                 |
| The writer is killed between create and rename          | No temporary file is made      | The temporary file stays; the target is as it was                  |
| The file starts with a byte order mark                  | Kept                           | Kept                                                               |

In every other case the file content, the set of files, and the exit code are those of the reference tool. The
permission bits of a target that exists are kept, as a write in place keeps them.

```text
known from the code alone:
    the temporary file is one the process creates, and only its permission bits are set, so its owner and group are
    the system's choice for a new file; an isolated tree has one user, and no test can show another
    an error of the lock call other than an interruption is returned as it is; which file systems refuse the call is
    not stated here, and no isolated tree has one
    a directory that can be written and searched and cannot be read: the open of the directory, on whose handle the
    lock is taken, fails before the target is read, so the call returns the system's error, no temporary file is
    created, and the target is unchanged; the writer ends as a configuration error naming the file, exit 1
    a target that is a mount point: the binary makes no test for one. It creates the temporary file beside the
    target and renames it over the target; when the rename fails, the temporary file is removed and the system's
    error is returned, so the writer ends as a configuration error naming the file, exit 1, the target unchanged
    a writer killed between create and rename: nothing removes `.<name>.<pid>.<n>.tmp`, and the target is unchanged
to be confirmed by a probe, each a unit test of files.rs:
    a_second_hard_link_of_the_target_keeps_the_old_content
        the other name reads the old content after the update, and the path the new content
    a_target_in_a_directory_that_takes_no_new_file_is_refused_and_left_as_it_was
    a_target_in_a_directory_that_cannot_be_read_is_refused_and_left_as_it_was
        the last two return early when the user may write any file
```

## Open questions

- That the system refuses the rename over a mount point is read in no source, and no check of this epic shows it: a
  mount needs a privilege no test has. Should the system allow the rename there, the file is replaced as any other and
  the row of the table above does not apply.
- Whether the reference tool reads a configuration file that starts with a byte order mark is read in no source. The
  binary's parser skips a leading mark and reads the file. Should the reference tool refuse such a file, the text is one
  more of `decision:parallel-workspaces/workspace-configuration/toml-dialect`, to be listed in
  [discovery-and-loading.md](./discovery-and-loading.md), and the last row of the table above does not hold for the
  reference tool. Nothing is decided here.
- That the standard library's exclusive file lock is held per open handle, makes a second handle wait, and ends when the
  handle is unlocked or closed is shown by component tests of `repo remove`. Which system call it is on each platform is
  not stated, and nothing depends on it.
- That a rename within one directory replaces the target atomically on the file systems the binary supports rests on the
  POSIX definition of `rename`; no probe can show it.
