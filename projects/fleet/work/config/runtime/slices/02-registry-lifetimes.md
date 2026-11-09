---
name: registry-lifetimes
---

# Lifetimes in the registry

Gives every key of both registries its lifetime and its `env`, and carries both into the contract documents
([lifetimes.md](../spec/lifetimes.md), [settings.md](../spec/settings.md)). Delivers `product:declared-lifetime`,
`product:boot-lifetime`, and the deprecation in `product:tick-interval-setting`.

## Change

- `Lifetime`, `ConfigKey.lifetime`, and `ConfigKey.env`; every key of both registries given its lifetime from
  [settings.md](../spec/settings.md); the variables that supply a key declared; both contract documents gain the two
  fields.
- The hub's `work_source[]` declared `deprecated`.
- The host file's `tick_seconds` key, `pass`, read at boot for now. `BZ_RUNNER_TICK_SECONDS` deprecated and honored only
  when the file states none. Both drivers take an interval callable.
- Every test and script that sets `BZ_RUNNER_TICK_SECONDS` moves to the file key in the same change: the crash suite's
  support (`tests/crash/support.py:59`), the runner and journey service specs, the two store-migration service tests,
  the rehearsal installer, the runner-host harness check, the step-pass component test, and the OpenCode bundle probe.
- `docs/versioning.md` §What counts as breaking gains the lifetime row, and `blizzard:compat` reports a key whose
  lifetime comes later than in a supported release's document ([lifetimes.md](../spec/lifetimes.md) §The contract
  carries it).
- Context: `bzh:config-key-declared` names the lifetime and the `env` rule (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test`: a registry refuses a value key with no lifetime, a grouping key with one, and a non-`boot` key
  with `env`.
- `blizzard:compat` against a fixture document moving a key from `pass` to `boot` reports a break.
- `blizzard:config-contract`, `blizzard:harness-check`, and `blizzard:crash-sweep` on the file key pass.
