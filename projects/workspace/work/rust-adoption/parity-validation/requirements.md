# Replacement confidence requirements

For `epic:rust-adoption/parity-validation`.

## Account for the old capabilities (product:capability-accounting)

The engineer can review the replacement's coverage against the existing Workspace rather than relying on a claim of
parity. `decision:rough-parity` allows deliberate redesign without concealing a missing capability.

Acceptance:

- Every inventoried command family and non-command capability has an owning epic and an observable acceptance outcome.
- Each existing capability is identified as preserved, deliberately changed, or awaiting a named decision; unresolved
  differences remain visible before adoption.
- The assessment includes safety refusals, partial failures, repeated reconciliation, and concurrent environments, not
  just successful single-environment examples.
- A claim of rough parity requires accepted evidence for every mapped capability or an explicitly approved replacement;
  an unresolved blocking decision or unimplemented capability cannot be counted as covered.

## Try representative work safely (product:isolated-adoption-proof)

The engineer can evaluate the replacement with disposable workspaces before authorizing it against their real work.

Acceptance:

- The evaluation covers repository and feature-environment setup, provisioning, provider lifecycle, generated agent
  tooling, nested workspaces, and operator inspection in representative declared configurations.
- Refusal and interruption examples show that unrelated work and resources survive.
- Results distinguish unsupported prerequisites from a failed capability, and include enough detail to reproduce a gap.
