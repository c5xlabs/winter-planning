# Nested workspace decisions

## Recursive teardown policy (decision:parallel-workspaces/nested-workspaces/recursive-teardown)

**Decider:** product owner and engineering

Outer teardown reads every child before any teardown, attempts every child env, and forwards one `--force` (and
`--strict` and `--no-provision-teardown`) at every depth, as the reference does; `--force` bypasses the dirty and
unpushed refusals. One deliberate fix: the reference goes on under `--force` past an unreadable child, a child env that
could not be torn down, or a child whose workspace services could not be stopped, and here any of them keeps the outer
worktree, even under `--force`, and the result says why.

## Delegation identity and ownership (decision:parallel-workspaces/nested-workspaces/delegation-policy)

**Decider:** product owner and engineering

Each init splits the parent's `ports_per_env` into K+1 equal slices for its K nested children, slice 0 the parent's and
the rest in configuration order, names each child `<service_prefix>-<env>-<repo>`, rewrites delegated settings every
init without retracting any, and inherits `["git"]` when `inherit_local` is unset, as the reference does. One deliberate
fix: an init that would move an existing child's slice names the children that move before applying it, rather than
relocating them silently.

## Child component compatibility (decision:parallel-workspaces/nested-workspaces/child-compatibility)

**Decider:** product owner, Platform engineering, and Workspace engineering

The child component is the `winter` found on `PATH` at the child root, and the protocol is its `ws status --json` at
`schema_version` 1, accepted on exit 0 or 1 when `root_path` equals the child root; each root is verified once per
process, and recursion is bounded by the nested chain alone, as the reference does. One deliberate fix: a child result
missing `stashes` or `local_only_commits`, or leaving out a standalone the child declares and holds on disk, counts as
unknown, not zero, so a child that cannot report its work never reads as having nothing to lose.
