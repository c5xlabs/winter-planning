---
epic: component-install
refinement: outline
status: planned
---

# Winter component install

A standalone capability epic supporting `epic:launcher-shim` and the Workspace and Fleet releases; its product
requirements live in [requirements.md](./requirements.md).

## The problem

The `winter` command runs only component versions already in the shared version cache, and nothing puts them there. A
person whose workspace pins a Workspace release, or who wants a newer hub or runner, has no supported way to obtain that
release; they are told it is missing and left to assemble the cache by hand.

## The outcome

A person can ask `winter` to fetch a published Workspace, hub, or runner release for their machine, or every release
their current selections need, and then use it through the normal entry point. A fetched release is checked before it
becomes usable, a failed fetch leaves nothing half-installed, and fetching never changes which version a workspace or
role selects. Running a command stays offline and predictable: an uncached version is still reported, now with the
command that would fetch it.

## Left for later

- Setting a workspace pin or a role's default version by command; selections stay hand-edited until a later epic.
- Removing cached releases that no selection uses.

## What this epic is not

- Publishing releases or defining their packaging; `epic:release-automation` and each product's release work own that.
- Fetching a missing version automatically when a command runs.
- Updating the `winter` executable itself, which Homebrew installs.
- Building a component from source.

## Depends on

- `epic:launcher-shim`, for the version cache layout and the selections fetching reads.
- `epic:release-automation`, for published releases to fetch.
