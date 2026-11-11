# Environment lifecycle decisions

## Environment identity and discovery (decision:parallel-workspaces/environment-lifecycle/discovery-authority)

**Decider:** product owner and engineering

**Open.** Choose the authority and repair workflow for registered names, on-disk directories, Git registrations, and
legacy manually created environments, including empty partial envs; this blocks finalizing `product:environment-targets`
and `product:environment-state-recovery` without inheriting three inconsistent discovery rules.

## Allocation policy and changes (decision:parallel-workspaces/environment-lifecycle/allocation-policy)

**Decider:** product owner and engineering

**Open.** Choose fixed-slot aliases, dynamic capacity, reservations, and migration when aliases or port sizing change,
including detection of host-level conflicts; this blocks finalizing `product:stable-environment-allocation`, not stable
ownership and cross-process collision protection.

## Teardown preservation and failure policy (decision:parallel-workspaces/environment-lifecycle/teardown-policy)

**Decider:** product owner and engineering

**Open.** Choose residual-file retention, local-history evidence, strict versus best-effort teardown, and safe release
after partial removal; this blocks finalizing `product:destroy-environments` and `product:environment-recovery`, with
resource and service action semantics owned by their respective parity epics.
