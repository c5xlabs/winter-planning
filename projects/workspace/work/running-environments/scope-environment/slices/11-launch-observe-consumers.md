---
name: launch-observe-consumers
depends-on:
  - compute-print
---

# The launch and observe consumers

Adds `Consumer::Launch`, `Consumer::Observe`, and `Computed::Degraded` of [consumers.md](../spec/consumers.md): the
`Launch` and `Observe` rows of Failure by consumer and of When lookups run. `Launch` runs lookups and refuses every
`EnvError`; `Observe` runs none, warns on an `IndexFault` and uses the recorded index, and answers the managed variables
alone, with a warning, when the bands fail. Nothing in this epic's milestone calls them:
`epic:running-environments/service-orchestration` calls them from its fan-out. Delivers the refusal and degraded bullets
of `product:complete-launch-environment`, the observe half of `product:offline-env-inspection`, and
`engineering:complete-resolution-outcomes`, under
`decision:running-environments/scope-environment/incomplete-launch-refusal`, a registered deviation for both.

## Change

- `Consumer` gains `Launch` and `Observe`, and `Computed` gains `Degraded { managed, cause }`.
- `Launch`: `Lookups::Run`; `UnknownScope`, `RecordedIndex`, and every resolution or lookup failure are `Err`.
- `Observe`: `Lookups::Mask`; `UnknownScope` is `Err`; an `IndexFault` writes one warning naming the scope, the index,
  and the fault, and resolution goes on; a resolution failure writes one warning naming the scope and the cause and
  answers `Degraded` with the managed variables of `scope-identity`.
- Warnings name the scope, band, and key, never a value.

## Acceptance

- Unit tests with a recording program port: `Launch` of a scope with a command entry starts it; `Observe` of the same
  scope starts nothing and answers `PLACEHOLDER` for it.
- Unit tests, `Launch`: a template `Token` error, a `ShellReference`, a failing lookup, a recorded index at the buffer
  slot, and an unrecorded name are each `Err`, and no `ScopeEnv` is answered.
- Unit tests, `Observe`: a recorded index at 0 writes one warning and answers `Complete` with `WINTER_ENV_INDEX=0`; an
  undefined reference writes one warning naming the scope and the key, and answers `Degraded` whose managed variables
  hold the scope's identity and port base; an unrecorded name is `Err(UnknownScope)`.
- A unit test: no warning text holds a value of the scope's bands.
- `cargo dev gate slice` exits 0.

## Out of scope

- Reporting a refused launch as a failed cell, the fan-out's reuse of one result per dispatch, and the provider call's
  environment, which removes the managed names of `scope-identity`, belong to
  `epic:running-environments/service-orchestration`, as do `restart` and `logs`, which compute no scope variables.
- The teardown computation of `ws destroy` belongs to `epic:running-environments/resource-provisioning`; what it does
  for an unrecorded environment is the open question of consumers.md, which environment-lifecycle engineering answers
  with that epic.
