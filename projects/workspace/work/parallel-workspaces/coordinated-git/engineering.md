# Coordinated Git engineering requirements

For `epic:parallel-workspaces/coordinated-git`.

## What already holds

- `ws connect`, `ws disconnect`, `ws diff`, `ws fetch`, `ws pull`, `ws merge`, and `ws push` write the drift warning of
  [drift-warning.md](../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md), which this epic
  wires with each command; `ws status` reads `drift::detect` and writes no notice. The catalog builds both functions.
- `ws status` runs the worktree probe of
  [location-listing.md](../../../delivered/parallel-workspaces/repository-catalog/spec/location-listing.md), The
  worktree probe, built by `epic:parallel-workspaces/repository-catalog` with `ws worktrees --status`; this epic's spec
  links it and adds its own pieces.
- The standalone integration of `ws pull` hands each standalone that declares a `ref` to `advance`, and `ws status`
  reports the pins of `snapshot`, Advance a pin on pull and Pin snapshot in
  [standalone-pins.md](../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md), built by
  `epic:parallel-workspaces/reproducible-inputs`; this epic implements `IntegrateCheckout` for `advance` and renders
  both.

## Shared Git-store coordination (engineering:shared-git-effects)

Operations account for linked worktrees sharing refs and configuration, including conflicting invocations from separate
processes. Refresh count, concurrency, and per-target order follow `engineering:bounded-fan-out`. Serves
`product:integrate-selected-repositories` and `product:push-worktree-destinations`.

## Bounded remote operations (engineering:remote-operation-bounds)

Remote operations have finite, documented deadlines, and their concurrency follows `engineering:bounded-fan-out`; only
classified transient failures retry within a bounded budget, while authentication and ref refusals retain their actual
cause. Overrides cannot disable limits accidentally through invalid values. Serves
`product:integrate-selected-repositories`.

## Native Git state fidelity (engineering:git-observation-fidelity)

State and diff collection preserve filenames containing whitespace or delimiters, renames, partially staged files, and
unmerged paths without relying on translated human status prose. A missing tracking ref remains distinct from missing
tracking configuration. Serves `product:inspect-workspace-git` and `product:worktree-upstream-control`.

## Preserve integration cleanup evidence (engineering:integration-recovery-evidence)

A failed `--abort` after a conflict and a failed autostash restore are their own outcomes, carrying the Git step that
failed, never a divergence. A completed remote effect, such as a refresh or a push, keeps its identity when later local
work in the same target fails. Serves `product:integrate-selected-repositories` and
`product:push-worktree-destinations`.
