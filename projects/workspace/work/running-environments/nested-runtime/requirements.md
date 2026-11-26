# Nested runtime requirements

For `epic:running-environments/nested-runtime`.

## Stop child runtime before removing its stores (product:nested-runtime-teardown)

The engineer can retire an outer environment without abandoning running child services or provisioned child resources.
Failure policy follows `decision:parallel-workspaces/nested-workspaces/recursive-teardown`.

Acceptance:

- Preview identifies child workspace-service shutdown and child provision teardown alongside the structural steps of
  `product:parallel-workspaces/nested-workspaces/nested-teardown-preservation`, without running mutation commands.
- Child workspace-service shutdown and provision teardown precede removal of the outer worktree.
- A child with no service binding does not fail solely because an inapplicable shutdown command was attempted.
- A failed child shutdown or provision teardown retains the outer store unless explicit, scoped approval permits
  proceeding, and the failure stays visible.
- Provision-teardown choices reach the child without broadening scope or bypassing root verification.

## Agree with the delegated allocation (product:nested-runtime-values)

The engineer can trust that a running child uses the namespace and ports its parent delegated.

Acceptance:

- Runtime inspection in the child agrees with the allocation delegated by the parent.
- A child whose runtime values fall outside its delegation is reported before its services start.
