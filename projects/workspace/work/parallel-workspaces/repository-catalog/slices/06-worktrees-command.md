---
name: worktrees-command
depends-on:
  - list-command
code:
  - winter-workspace/crates/core/src/state_registry.rs::read
---

# `ws worktrees` without `--status`, and the environments present

Adds the `ws worktrees` command of [location-listing.md](../spec/location-listing.md) for its locations, its document
without the three status keys, and its human rows, with its first caller's piece of the seam: Environments present of
the same leaf. Delivers the location-listing bullet and the location-listing half of the creates-nothing bullet of
`product:inventory-presence`, and the workspace mark of `product:repository-roles`. The environment rule is the
reference's, and `decision:parallel-workspaces/environment-lifecycle/discovery-authority` owns any change to it.
`--status` follows in `worktree-status`, under `decision:parallel-workspaces/repository-catalog/direct-landing`.

## Change

- `state_registry` gains `read_in_order`, the recorded names and values in the state file's order, with the refusals of
  `read`: a file that cannot be read, is not UTF-8, or does not parse is `ConfigError::FileRead` naming it, under
  `decision:parallel-workspaces/workspace-configuration/state-read-failures`. `read` becomes the map of `read_in_order`;
  `env_index::lookup` and `index_lookup::answer` stay as they are, and `present` builds the map `lookup` takes from that
  list.
- `crates/core/src/environments.rs` holds `present` over the loaded `WorkspaceConfig`: `ports.env_aliases` in order,
  then the recorded names in the state file's order, a repeated name dropped; a candidate is present when
  `<root>/<name>` is a directory and one of its entries is a directory named as the `common.name` of an entry of
  `project_repositories`, both with symbolic links followed; the present names sorted by the index `lookup` answers,
  stably. A candidate directory that cannot be listed is `InternalError::Unexpected`.
- `crates/core/src/commands/ws/worktrees.rs` registers `ws worktrees` with the arguments of its row of
  [command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands,
  `--status` apart: until `worktree-status` lands it is an unknown option, exit 2.
- `crates/core/src/locations.rs` lists, after the load and the read of the state file, a worktree location for each
  environment present and each loaded project entry, then each standalone, then the workspace, each kept only when it is
  a directory, symbolic links followed, with the kinds, labels, names, and paths of the spec.
- `Location` is built without its `status` field and `ListError` without `Repo`; both arrive with `worktree-status`,
  their first users.
- The command writes the spec's Shape A array, the three status keys absent, or one human row per location with the
  label, the kind, and the path; it writes no drift notice, starts no program, and writes no file.
- `environments` joins `COMMAND_FILE_NAMES` in `crates/dev/src/architecture.rs`, its pinned test renamed for the new
  count; `locations` does not, since the command file calls it.
- `context/architecture/structure.md` and `changes.md` gain the rows for `environments.rs`, `locations.rs`, and the
  command file.
- Recorded scenarios under `scenarios/ws-worktrees/`, recorded with `cargo dev record` from the reference tool, the
  executable path given with the task, `stdout = "json"`: a workspace with no environment; two remotes with checkouts
  under `projects/` and worktrees in `alpha/` and in a recorded environment with a higher index, one project's worktree
  missing from `alpha/`; a declared standalone present and one absent; a directory `lambda/` holding a project
  directory, not listed; a pinned project; a state file holding `[env_index` left open, which the reference reads as
  empty; its `[deviation]` table names `decision:parallel-workspaces/workspace-configuration/state-read-failures`, and
  `expected/` holds exit 1 and no document.

## Acceptance

- `cargo scenarios ws-worktrees` passes every scenario, and `cargo scenarios ws-index` passes unchanged; each
  `ws-worktrees` record's document ends with the `workspace` entry and has no `ahead`, `behind`, or `dirty` key.
- Unit tests of `present` with `MemoryFiles`: an alias directory holding `api/` is present and one holding only `other/`
  is not; a directory neither aliased nor recorded is not a candidate; a recorded name is ordered by its recorded index
  between aliases; two names with one index keep alias-then-registry order; a file named as an alias is not present; a
  candidate failing reads gives `InternalError::Unexpected`.
- Unit tests of `state_registry::read_in_order`: names come back in the file's order; a missing file is empty; a file
  that does not parse, and one that is not UTF-8, are `FileRead` naming the file; `read` answers as before, its existing
  tests unchanged.
- Component tests: an alias whose project entry is a symbolic link to a directory is listed; the human rows carry the
  labels in the document's order; the tree and the git state are unchanged after a run.
- `cargo dev gate full` exits 0.

## Out of scope

- `--status` and its probes: `worktree-status`. `ws list`: `epic:parallel-workspaces/environment-lifecycle`.
