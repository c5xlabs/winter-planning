# Repository catalog decisions

## Landing the catalog's changes (decision:parallel-workspaces/repository-catalog/direct-landing)

**Decider:** engineering

A change to the shipped `repo remove` lands directly in one slice, and a new command may land in steps across slices,
with no flag and no expand-and-contract: `winter-workspace` declares no breaking-change policy and the binary has no
user or caller outside its own tests and recorded scenarios, since it is never installed (its `AGENTS.md`, Standing
rules) and no release exists before `epic:rust-adoption/workspace-release`. Between steps a command may lack an option
it will have, a check it will run under the write lock, or its operation spans; each step keeps every recorded scenario
passing, or records the deviation that changes one.

## Edits the catalog refuses (decision:parallel-workspaces/repository-catalog/refused-edits)

**Decider:** product owner

`repo add` and `repo remove` refuse to edit a file whose repository array of the requested type is anything but an array
of tables, such as an inline array of tables, and `repo remove` refuses a line edit whose result does not parse to the
document less the matched block; each refusal exits 1 and leaves the file unchanged, so an edit never reaches a file the
line editor cannot change safely. Where the reference tool removes the matched element of an inline array, and where it
would remove a block the binary's line edit refuses, the configuration that results differs, so those two refusals are
registered deviations; the refusal on `repo add` is not one, because the reference also ends with exit 1 and the file
unchanged, and how it fails is its TOML library's (`decision:toml-parity`).

## Repository identities (decision:parallel-workspaces/repository-catalog/repository-identities)

**Decider:** product owner

Repository identities are the reference tool's: one repository may be declared both as a project and as a standalone,
and extension consumers then take the standalone entry; sources compare as exact strings, so `…/api` and `…/api.git` are
two sources; and the implicit repositories are the workspace root, `product/` when it is a directory, and
`context/harness/` when it holds `.git`. Which declarations act as extensions is
`decision:agent-ready-operation/extension-management/adoption-policy`; a stricter identity rule would refuse
configurations existing workspaces rely on.

## Managed locations (decision:parallel-workspaces/repository-catalog/location-policy)

**Decider:** product owner

Managed locations are the reference tool's: a standalone `path` is relative and has no `..` segment, a new standalone is
refused only when its location resolves, symlinks followed, to the resolved location of a declared standalone, and no
name is reserved and no symlink refused. Stricter confinement at the point of cloning belongs to the materialization of
`epic:parallel-workspaces/workspace-bootstrap`, the only consumer that writes into these locations.

## Comments removed with a repository (decision:parallel-workspaces/repository-catalog/removal-comment-extent)

**Decider:** product owner

`repo remove` removes the block and its own comments only: the comment lines directly above the removed block go with
it, and the comment lines directly above the next block are kept, because a comment directly above a block describes
that block. The reference tool does the reverse on both counts, leaving the removed block's comment behind and deleting
the next block's, so both halves are a registered deviation.
