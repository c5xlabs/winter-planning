# Nested workspace requirements

For `epic:parallel-workspaces/nested-workspaces`.

## Initialize only the intended child workspace (product:verified-child-workspace)

The engineer can mark a project as a nested workspace and initialize its environment checkout without running the
enclosing workspace accidentally.

Acceptance:

- Nested initialization runs only in the selected env worktree after its declared lightweight bootstrap succeeds.
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
- Adding, reordering, or removing children does not silently relocate live resources; affected changes and recovery are
  explained before application.
- Runtime inspection in the child agrees with the allocation delegated by the parent.

## Inherit only approved local settings (product:nested-local-inheritance)

The engineer can share selected local settings with a child without copying the enclosing workspace's private repository
list, secrets, runtime selection, or unrelated configuration by accident.

Acceptance:

- The inherited set is explicit and inspectable; disabling inheritance leaves existing child-owned settings intact.
- Only named keys from the parent's actual local layer are copied, not every effective default or committed setting.
- Repository declarations are not copied by a general inheritance request.
- Parent delegation wins conflicts with inherited allocation keys; unrelated child content and comments remain intact.
- Malformed input or a footprint refusal leaves the child overlay unchanged.
- Removed inherited or delegated settings follow ownership policy rather than deleting indistinguishable user work.

## Inspect recursive work without hiding unknown state (product:nested-work-inspection)

The engineer can see which child environments and stores hold dirty or unpublished work before deciding to remove the
outer environment.

Acceptance:

- Inspection reports child identity, environment count, dirtiness, and work that would be lost, recursively.
- Source clones, standalones, detached work, retained branches, stashes, and env worktrees are within the loss audit.
- A branch held remotely but not merged is distinguishable from one whose commits exist only inside the child.
- Failed or incomplete child inspection remains an unknown safety condition, not a healthy empty workspace.
- Child errors are attributed to the outer repository and do not suppress other child results.

## Retire child resources before removing their stores (product:nested-teardown-preservation)

The engineer can retire an outer environment without abandoning running child services or deleting the only copy of
child work. Failure policy follows `decision:parallel-workspaces/nested-workspaces/recursive-teardown`.

Acceptance:

- Dirty, unpublished, unreadable, or incompletely audited child work refuses default outer teardown before destruction.
- Preview identifies nested env teardown, child workspace-service shutdown, and loss-bearing store removal without
  running mutation commands.
- Child env lifecycle teardown and child workspace-service shutdown precede removal of the outer worktree.
- A child with no service binding does not fail solely because an inapplicable shutdown command was attempted.
- Child failure retains the outer store unless explicit, scoped discard/teardown approval permits proceeding; failure
  and retained-work outcomes remain visible even when approved cleanup continues.
- Strictness and provision-teardown choices reach the child without broadening scope or bypassing root verification.
