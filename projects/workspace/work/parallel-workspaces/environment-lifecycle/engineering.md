# Environment lifecycle engineering requirements

For `epic:parallel-workspaces/environment-lifecycle`.

## What already holds

- The rule that finds the environments present is specified in
  [location-listing.md](../../../delivered/parallel-workspaces/repository-catalog/spec/location-listing.md),
  Environments present, as the reference's, and is built with `ws worktrees` by
  `epic:parallel-workspaces/repository-catalog`.
  `decision:parallel-workspaces/environment-lifecycle/discovery-authority` keeps that rule for `ws list` and glob
  targets; `ws list` links it rather than copying it.
- `ws list` writes the drift warning of
  [drift-warning.md](../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md), which this epic
  wires with that command: the catalog builds `drift::detect` and `drift::notice_line`, and this epic's spec says where
  in the run the notice is written.
- Repository fan-out in `ws init` and the serial order of `ws destroy` follow `engineering:bounded-fan-out`.
- `ws index` is built, with its recorded scenarios, by `epic:rust-adoption/parity-validation`, as
  [index-lookup.md](./spec/index-lookup.md) of this epic gives it; it answers an allocated slot as `registry` and a
  hypothetical one as `suggested` and reserves nothing, as `product:stable-environment-allocation` asks. This epic
  leaves it unchanged: `allocate` is added beside `lookup` in `env_index.rs`, and no second lookup is built.

## Cross-process allocation integrity (engineering:allocation-integrity)

Assignment, collision resolution, and release remain consistent across processes and interruption; independent readers
cannot observe a successful duplicate assignment. A state file that cannot be read ends an allocation-changing command
before any effect. Serves `product:stable-environment-allocation` and `product:environment-state-recovery`.

## Verify worktree identity at the effect boundary (engineering:worktree-effect-confinement)

Creation and removal verify the intended workspace ancestry, Git common store, and worktree registration, including path
replacement between planning and execution, so no effect lands outside the env directory that
`decision:parallel-workspaces/environment-lifecycle/env-name-grammar` admits, as
`decision:parallel-workspaces/environment-lifecycle/location-confinement` decides. Serves `product:environment-targets`
and `product:environment-worktrees`.

## Preserve independent bootstrap outcomes (engineering:lifecycle-step-isolation)

Tracking-wiring failure remains attributable without suppressing independent lightweight bootstrap work; automatic Git
tracking defaults cannot override the env's intended tracking. Repository sequences and allocation lifetime remain
recoverable after process failure. Serves `product:environment-worktrees` and `product:environment-recovery`.

## Safety probes cannot degrade to empty state (engineering:teardown-evidence-completeness)

Destructive preflight distinguishes absent data from failed inspection; a failed inspection is never read as nothing at
risk. Read-only preview cannot run teardown commands or repair registration implicitly. Serves
`product:destroy-environments` and `product:environment-state-recovery`.
