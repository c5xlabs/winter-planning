---
name: compute-print
depends-on:
  - scope-identity
  - resolve-command-entries
  - run-lookup
---

# The one computation entry, for printing

Adds `compute` and `Consumer::Print { resolve }` of [consumers.md](../spec/consumers.md), Domain: `identity`, then
`resolve` with `Lookups::Run` when `resolve` is set and `Lookups::Mask` otherwise, its `run` callback bound to
`run_lookup` over the invocation's program port, interrupts, and root. Builds the `Print` row of Failure by consumer and
the `winter env` and "one dispatch" rows of When lookups run. Delivers `engineering:scope-computation-authority` and
`engineering:complete-resolution-outcomes` for printing, and the no-cache bullet of `product:external-value-resolution`,
under `decision:running-environments/scope-environment/lookup-execution-policy`.

## Change

- `scope_env.rs` gains `Consumer` with `Print`, `Computed` with `Complete`, and `compute`; under `Print`,
  `UnknownScope`, an `IndexFault` as `EnvError::RecordedIndex`, and a resolution or lookup failure are each `Err`, and
  nothing is written to its `Diagnostics`.
- `compute` keeps nothing between calls: each call runs its lookups again.
- No command calls it yet.

## Acceptance

- Unit tests over a loaded workspace and a recording program port: `Print { resolve: false }` of a scope with a command
  entry starts no program and answers `PLACEHOLDER` for it; `Print { resolve: true }` starts it once and answers its
  value.
- A unit test: two `compute` calls for two feature scopes, each with `resolve`, start the workspace band's lookup twice.
- Unit tests: an unrecorded name is `Err(UnknownScope)`; a name recorded at the buffer slot is `Err(RecordedIndex)` with
  `Buffer`; a failing lookup is `Err(Resolution(Lookup))` naming its entry.
- A unit test: nothing is written to the `Diagnostics` in any of these cases.
- `cargo dev gate slice` exits 0.

## Out of scope

- `Launch`, `Observe`, and `Computed::Degraded` land with `launch-observe-consumers`.
