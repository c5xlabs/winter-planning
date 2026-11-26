---
name: harness-set
external: github:paul-gross/blizzard#2128
code:
  - blizzard/src/blizzard/runner/harness/wiring.py::select_harness_bindings
  - blizzard/src/blizzard/runner/harness/wiring.py::publish_harness_bundle
  - blizzard/src/blizzard/runner/harness/wiring.py::build_production_harnesses
  - blizzard/src/blizzard/runner/composition.py::_build_runner
  - blizzard/src/blizzard/runner/harness/claude_code/adapter.py::ClaudeCodeAdapter
  - blizzard/src/blizzard/runner/harness/opencode/adapter.py::OpenCodeAdapter
  - blizzard/src/blizzard/runner/harness/harness_telemetry_plan.py::HarnessTelemetryPlan
  - blizzard/src/blizzard/runner/harness/capability_snapshot.py::HarnessVersionCache
  - blizzard/src/blizzard/runner/lifecycle/spawn.py::Spawner
  - blizzard/src/blizzard/runner/loop_wiring.py::RunnerStepper._context
---

# The harness set and settings at launch

Gathers each runner's harness bindings, bundle, health probes, and telemetry plan into a `HarnessSet` built from the
`session` keys it reads. Each launch is handed a `SessionSettings` and holds the set it launched with
([passes.md](../spec/passes.md) §Sessions, §The objects settings reach, §The harness set, §A runner's tick). The
runner's behaviour does not change: the file is still its only source, so a runner builds one set at composition, as it
does today. Pre-work for `product:session-settings` and `product:session-lifetime`, bound by
`engineering:running-session-undisturbed`.

## Change

- `HarnessSet` and `HarnessSets` in `runner/harness/harness_set.py`. `select_harness_bindings`,
  `publish_harness_bundle`, and `build_production_harnesses` are called through them, and a set is rebuilt when its key
  differs from the last one's.
- `HarnessVersionCache` keyed by binary path, size, and modification time.
- `SessionSettings` built per launch. The adapter's launch composes the worker's command, environment, and settings from
  it alone, and each running session is supervised through the set it launched with.
- A launch is handed the bundle snapshot's own path, never the `current` link.
- `RunnerStepper._context` builds under `_build`, the loop's hub client keyed by its timeout and closed only once both
  step locks are idle.

## Acceptance

- `blizzard:unit-test`: a set key that does not move builds nothing; a moved binary is probed again and an unmoved one
  is not; an unchanged bundle publishes to the snapshot it already has.
- `blizzard:component-test`: a session launched under one set keeps its bundle snapshot and binding after another set is
  built, and the old set is dropped when its last session ends.
- `blizzard:component-test`: a moved hub timeout closes the old client only after a trace sweep holding it ends.
- `blizzard:harness-check`, `blizzard:service-test`, and `blizzard:crash-sweep` pass with no test changing its
  expectations.
