---
name: registry-lifetimes
external: github:paul-gross/blizzard#2126
code:
  - blizzard/src/blizzard/foundation/config_keys.py::ConfigKey
  - blizzard/src/blizzard/foundation/config_keys.py::ConfigRegistry
  - blizzard/src/blizzard/tools/config_contract.py::document_from_registry
  - blizzard/src/blizzard/runner/loop_wiring.py::PeriodicDriver
  - blizzard/src/blizzard/runner/loop/periodic_pass_driver.py::PeriodicPassDriver
---

# Lifetimes in the registry

Gives every key of both registries its `lifetime` and `env`, carries both into the contract documents, and adds
`tick_seconds` as a file key ([lifetimes.md](../spec/lifetimes.md) §Domain, §Interfaces;
[settings.md](../spec/settings.md) §Data, §Pinned). Delivers `product:declared-lifetime`, `product:boot-lifetime`,
`product:host-footing`, and the deprecation in `product:tick-interval-setting`; `engineering:no-renamed-settings` holds
every key's name.

## Change

- `Lifetime`, `ConfigKey.lifetime`, and `ConfigKey.env`; `ConfigRegistry`'s three refusals; every key of both registries
  given its lifetime from [settings.md](../spec/settings.md) §Data, and none declared for a constant §Pinned lists; the
  four variables of [lifetimes.md](../spec/lifetimes.md) §Environment variables declared.
- Both contract documents and `contracts/config/README.md` gain the two fields.
- The hub's `work_source[]` declared `deprecated`, `boot`, `replaced_by` the work-source records.
- `tick_seconds`, `pass`, read from the file at boot until [06-host-settings](./06-host-settings.md) makes it an entry.
  `BZ_RUNNER_TICK_SECONDS` deprecated and honored only when the file states none. Both drivers take
  `interval: Callable[[], float]` and wait from the pass's end, as today, with a stop ending the wait
  ([passes.md](../spec/passes.md) §The loop's interval).
- Every test and script setting `BZ_RUNNER_TICK_SECONDS` moves to the file key in the same change: the crash suite's
  support (`tests/crash/support.py`), the runner and journey service specs, the two store-migration service tests, the
  rehearsal installer, the runner-host harness check, the step-pass component test, and the OpenCode bundle probe.
- `docs/versioning.md` §What counts as breaking gains the lifetime row; `blizzard:compat` reports a lifetime moved later
  as a break (`lifetime_breaks`, [lifetimes.md](../spec/lifetimes.md) §Contract documents).
- Context: `bzh:config-key-declared` names the lifetime and the `env` rule (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test`: a registry refuses a value key with no lifetime, a grouping key with one, and a non-`boot` key
  with `env`.
- `blizzard:compat` against a fixture document moving a key from `pass` to `boot` reports a break.
- `blizzard:unit-test`: neither registry declares a key for a constant [settings.md](../spec/settings.md) §Pinned lists.
- `blizzard:unit-test`: a driver waits the full interval after a pass that ran longer than it, and a stop ends its wait
  at once.
- `blizzard:config-contract`, `blizzard:harness-check`, and `blizzard:crash-sweep` on the file key pass.
