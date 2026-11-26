# Nested workspace requirements

For `epic:parallel-workspaces/nested-workspaces`.

## Initialize only the intended child workspace (product:verified-child-workspace)

The engineer can mark a project as a nested workspace and initialize its environment checkout without running the
enclosing workspace accidentally.

Acceptance:

- Initializing the outer env runs the nested project's declared bootstrap, verifies the child, writes its delegated
  settings, then runs a bare `winter ws init` in the child, which clones its projects and creates no child feature envs.
- `ws init --all` repeats this for every outer env.
- The project's source checkout is not initialized as a child workspace, and nesting does not imply extension execution.
- Before child mutation, the caller verifies the child's workspace root and usable inspection result.
- A missing marker, mismatched root, unsupported child, or recursion into an ancestor refuses without child mutation.
- A misresolved child cannot recursively fan out across sibling nested workspaces.
- The selected child component and any compatibility refusal are visible under
  `decision:parallel-workspaces/nested-workspaces/child-compatibility`.

## Delegate non-overlapping child allocations (product:nested-runtime-allocation)

The engineer can run sibling nested workspaces without them sharing the parent's service namespace or claiming each
other's allocated ports. Assignment policy follows `decision:parallel-workspaces/nested-workspaces/delegation-policy`.

Acceptance:

- Each child receives a distinguishable namespace and an allocation contained within the parent's reservation.
- The parent's own services retain capacity, and a child's full shared-plus-feature footprint must fit its allocation.
- An oversized or zero-capacity allocation refuses before writing child settings or initializing the child.
- Allocation uses the parent's actual persisted assignment, not a hypothetical hash suggestion.
- An init that would move an existing child's allocation names the children that move before applying it.
- The child's own environment inspection reports the namespace and ports delegated by the parent.

## Inherit only approved local settings (product:nested-local-inheritance)

The engineer can share selected local settings with a child without copying the enclosing workspace's private repository
list, secrets, runtime selection, or unrelated configuration by accident.

Acceptance:

- With no inherited set configured, only the parent's local `git` settings are copied; disabling inheritance leaves
  existing child-owned settings intact.
- Only named keys from the parent's actual local layer are copied, not every effective default or committed setting.
- Repository declarations are not copied by a general inheritance request.
- Parent delegation wins conflicts with inherited allocation keys; unrelated child content and comments remain intact.
- Malformed input or a footprint refusal leaves the child overlay unchanged.
- An inherited or delegated setting the parent no longer supplies stays in the child as it was.

## Inspect recursive work without hiding unknown state (product:nested-work-inspection)

The engineer can see which child environments and stores hold dirty or unpublished work before deciding to remove the
outer environment.

Acceptance:

- Inspection reports child identity, environment count, dirtiness, and work that would be lost, recursively.
- `ws status --json` gives each nested-workspace worktree a `nested` object with its env count, dirty and unpushed
  flags, and any error; an unreadable child sets the error and counts as unknown.
- Source clones, standalones, detached work, retained branches, stashes, and env worktrees are within the loss audit.
- A branch held remotely but not merged is distinguishable from one whose commits exist only inside the child.
- Failed or incomplete child inspection remains an unknown safety condition, not a healthy empty workspace.
- Child errors are attributed to the outer repository and do not suppress other child results.

## Retire child workspaces before removing their stores (product:nested-teardown-preservation)

The engineer can retire an outer environment without deleting the only copy of child work or leaving a child's services
and provisioned resources behind. Failure policy follows
`decision:parallel-workspaces/nested-workspaces/recursive-teardown`.

Acceptance:

- The loss audit is taken before any child teardown begins.
- Dirty, unpublished, unreadable, or incompletely audited child work refuses default outer teardown before destruction.
- Preview identifies each nested env teardown, each child workspace-service stop, and loss-bearing store removal without
  running mutation commands.
- Each child env is retired by the child's own `ws destroy`, which runs its provision teardown and stops its env
  services; the child's workspace services are then stopped by the child's own `service down workspace`.
- Child teardown completes before any outer teardown effect, including the outer provision teardown and worktree
  removal.
- A child with no service binding does not fail because a workspace-service stop does not apply to it.
- An unreadable child, a failed child env teardown, or a failed child workspace-service stop keeps the outer
  environment, even under `--force`, and the result names the child and env that failed and that the outer environment
  was kept.
- The outer `--force`, `--strict`, and `--no-provision-teardown` reach every child without broadening scope or bypassing
  root verification.
