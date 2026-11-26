---
name: allocation-rule
---

# The allocation decision and registry findings

Adds the pure half of [allocation.md](../spec/allocation.md), Domain, Allocate, and Findings: `env_index::allocate`,
which decides an environment's index from the aliases, the band, and the registry in file order, and
`env_index::findings`, which names every registry entry a command reports. With them, `EnvIndexError` with the
`BandFull` of the frozen
[lock-and-state.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md), Env index
allocation, and this epic's `RecordedReserved` and `AliasSlotHeld`. Delivers the decision rules of
`product:stable-environment-allocation` and the findings of `product:environment-state-recovery`, under
`decision:parallel-workspaces/environment-lifecycle/allocation-policy`, whose refusals and reports are registered
deviations.

## Change

- `crates/core/src/env_index.rs` gains `Allocation`, `allocate`, `RegistryFinding`, `Reserved`, `findings`, and
  `EnvIndexError`, beside the built `lookup`, whose answer is unchanged.
- `allocate` probes the hash band as lock-and-state.md gives it, from the registry passed in, and records nothing.
- No command calls either yet.

## Acceptance

- Unit tests, an alias: unrecorded is `New` at its position; recorded there is `Recorded`; recorded elsewhere is
  `Moved { from, to }`; with another name recorded at its position, `AliasSlotHeld` names the first such name in file
  order; an alias listed twice takes its later position.
- Unit tests, a name that is no alias: recorded at an alias's slot is `RecordedReserved` naming that alias; recorded at
  `N + 1` is `RecordedReserved` naming the buffer; recorded at `0`, at a negative index, beyond `M`, or at an index
  another name also records is `Recorded` at that index.
- Unit tests, a new name: `New` at `suggested(name)` when free; past each taken slot to the next, wrapping to the band's
  first slot; `BandFull` naming the band's size and bounds when every slot is taken.
- Unit tests of `findings`: one workspace holding every kind answers `OutOfRange`, `ReservedSlot`, `AliasMoved`, then
  `Shared`, each kind in file order, and `Shared` once per index with its names in file order; a clean registry answers
  none.
- `cargo dev gate slice` exits 0.
