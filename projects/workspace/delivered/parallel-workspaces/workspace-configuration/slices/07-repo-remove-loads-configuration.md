---
name: repo-remove-loads-configuration
depends-on:
  - load-seam
  - repository-entries
---

# `repo remove` refuses an invalid configuration and names a failed write

Puts the load of [discovery-and-loading.md](../spec/discovery-and-loading.md) before the edit of `repo remove` and gives
a failed replacement its own kind, under `decision:parallel-workspaces/workspace-configuration/write-failure-kind`. Adds
the probes of [write-path.md](../spec/write-path.md) for
`decision:parallel-workspaces/workspace-configuration/replacement-edges`, and the checks of
`decision:parallel-workspaces/workspace-configuration/write-byte-preservation`. Serves
`product:configuration-validation`, `engineering:configuration-snapshot`, and
`engineering:configuration-write-integrity`.

## Change

- `crates/core/src/repository_catalog.rs`: `remove` calls `config_load::load` where it calls `workspace_root::find`,
  after the shape of the target is checked and before the type word is checked, and takes the root from the `Workspace`;
  a load failure ends the command before any file is read for the edit. The edit still works on the bytes `update` hands
  it under the lock, not on the loaded value. An `Err` of `WriteFiles::update` becomes `ConfigError::FileWrite` with the
  path of the file.
- `crates/core/tests/repo_remove.rs`, four tests change, because the load now reads both files before the edit:
  - `a_file_that_cannot_be_opened_for_writing_is_a_file_read_error_and_is_left_as_it_is` asserts `FileWrite` and is
    renamed for it.
  - `the_document_holds_the_name_as_the_target_gives_it_not_as_the_block_does` holds `name = 7`, which the load refuses;
    it is replaced by a test that the removal is refused with `ValueType` and the file is left as it was.
  - `a_file_that_is_not_valid_toml_is_a_file_read_error_and_is_left_byte_equal` makes both files invalid and expects the
    error to name the file each flag edits. It becomes two cases: an invalid shared file beside a valid local file is
    `FileRead` naming the shared file, with and without `--local`; a valid shared file beside an invalid local file is
    `FileRead` naming the local file, with and without `--local`. Both files are byte equal afterwards in each.
  - `the_lock_is_released_after_a_removal_and_after_each_refusal` holds a local file that is not valid TOML, on which
    every run now ends at the load. Its local file becomes valid, and it asserts the lock free after a target that is
    not found, after an unknown type word, and after a removal; a second workspace with an invalid local file asserts
    that the removal ends as `FileRead` naming the local file and that the lock is free.
- `crates/core/src/files.rs` gains three unit tests of `OsFiles::update`, each in a temporary directory, named as
  [write-path.md](../spec/write-path.md) names them: `a_second_hard_link_of_the_target_keeps_the_old_content`;
  `a_target_in_a_directory_that_takes_no_new_file_is_refused_and_left_as_it_was`;
  `a_target_in_a_directory_that_cannot_be_read_is_refused_and_left_as_it_was`. The last two return early when the user
  may write any file, as the existing test of an unwritable target does, and restore the mode of the directory before
  the temporary tree is dropped. No code of `files.rs` outside its tests changes.
- Recorded scenarios, each plain, by `cargo dev record <id> --reference <path>` over a fixture workspace in a temporary
  directory: `repo-remove/invalid-configuration`, whose shared file holds `envs_per_workspace = 1` and one repository,
  and `repo-remove/invalid-local-configuration`, whose local file holds it.
- The built binary writes configuration files in this change. It is run only against fixture workspaces in temporary
  directories, never against a real workspace.
- Files touched: `crates/core/src/repository_catalog.rs`; `crates/core/src/files.rs`, tests only;
  `crates/core/tests/repo_remove.rs`; `scenarios/repo-remove/**` for the two scenarios.

## Acceptance

- A component test in `crates/core/tests/repo_remove.rs` runs `repo remove` over a workspace whose shared file holds
  `envs_per_workspace = 1` and asserts exit code 1 and both files byte for byte as before.
- A component test there removes one of two entries from a file whose lines end in CR LF and which holds a comment with
  a character of more than one byte, and asserts that the file afterwards equals the bytes before with the bytes of the
  removed block taken out.
- Unit tests in `repository_catalog.rs`, over doubles of the two file ports:
  - `remove` with a `WriteFiles` double whose `update` fails is `ConfigError::FileWrite` with the path of the file;
  - `remove` whose read port gives a file of two entries at the load, while the write port hands the edit the same file
    with a third entry, removes its own entry and keeps the third;
  - the existing test `a_file_that_is_not_valid_toml_is_a_file_read_error_on_the_edited_file` passes unchanged.
- The three unit tests of `files.rs` pass. If one fails as written, the change stops and reports what `update` did; it
  does not change `update`.
- `cargo scenarios repo-remove` passes: the two new scenarios show exit code 1 and an unchanged tree in the record of
  the reference tool, and every existing scenario replays with no change to a recorded file. If the reference tool
  removes the entry instead, the change stops and reports it.
- Every existing test of `repo remove` other than the four named passes unchanged, among them
  `the_command_waits_while_another_handle_holds_the_lock_of_the_directory`. `cargo dev gate slice` passes.

## Out of scope

- The write path itself, which is unchanged.
- A target that is a mount point: no test here can mount one, and the open question of the spec stands.
