---
name: host-scope-pilot
depends-on:
  - host-scope-cache
  - hub-request-pilot
code:
  - blizzard/src/blizzard/runner/composition.py::build_host_process
  - blizzard/src/blizzard/runner/composition.py::HostProcess
  - blizzard/src/blizzard/runner/composition.py::compose_runner
  - blizzard/src/blizzard/runner/composition.py::HostRunnerReconcile
  - blizzard/src/blizzard/runner/composition.py::build_runner_process
  - blizzard/src/blizzard/runner/composition.py::add_runner
  - blizzard/src/blizzard/runner/app.py::_wire_hosted_app
  - blizzard/src/blizzard/runner/loop_wiring.py::LoopWiring
---

# Host and runner child-scope pilot

Activates the host's locked lifetime graph from spec [Scope ownership](../spec/scopes.md), delivering its
`engineering:born-in-container`, `engineering:container-per-process` and `engineering:graph-checked-whole`. The hub
pilot's executable authenticated/background proof is the activation prerequisite under
`decision:adopt-di/strangler-migration`.

## Change

- Serve the running host from the host root built in `host-scope-cache`. Host routes get the host container through
  `host_request`, the router dependency that attaches it to the request, as the hub's `process_request` does. Each
  runner child initially retains named legacy leaves and runtime adapters, and a bad entry still fails the `compose` or
  `serve` start step.
- Construction paths:
  - Boot: `build_host_process` stops composing runners; `_wire_hosted_app`'s `HostRunnerSet` opens a child for each
    seeded runner, where it places today's `compose_runner` results.
  - `HostRunnerReconcile`: every start attempt, a retry included, opens a new child. `runner add` on a running host
    writes a hosted-runner record that the next tick reconciles, so it is not a separate path.
  - Standalone: `build_runner_process` builds a host root the caller owns, whose `HostRunnerSet` opens the one child;
    closing that runner closes the host root, as `closes_host` does today. Its callers are `LoopWiring.with_context`
    (the transcript verbs), `LoopWiring.tick_once` without a process, the CLI `tick` path in `runner/cli/runtime.py`,
    and `build_hosted_app`'s host-of-one. Each supplies the root its `HostConfig` and the child its `BuiltRunner`; the
    transcript verbs' `_transcript_config` already holds the host config. The CLI `tick` builds that one container, its
    command-local services at the host translation's `PROCESS` lifetime, and owns and closes it on exit, including
    exceptions ([Interfaces](../spec/scopes.md#interfaces)).
- Delete `composition.add_runner`, which only tests call (`engineering:dead-members-deleted`), moving its tests onto
  `HostRunnerSet`.
- Register a representative host leaf service, one depending on no runner child and not on `HostRunnerSet`, and consume
  it through a served host route. Host readiness is not one: it reads the halted runners `HostRunnerSet` holds.
- Retarget the runner hand-wiring guards in `tests/guards/test_layering.py` (`_COMPOSITION_ROOTS`,
  `_RUNNER_GRAPH_CONSTRUCTORS`, and the guard that only composition names a runner-store adapter) to the host root and
  area wiring modules.
- Land host transitional harness guidance, wiring exemplar and duplicate/identity/lifetime gates.
- Require new host/runner services to use area registration only after the usable pilot lands.

## Acceptance

- `blizzard:component-test`: the representative leaf service is served from the new graph; an exemplar/test-only service
  is registered and consumed through the same production resolution path without changing the root's individual service
  list.
- `blizzard:component-test`: draining one runner leaves its sibling serving and resolving; a start failing before or
  after placement leaves no open runner child and reports the same start step as today; the next reconcile opens a new
  one; a standalone runner's closure also closes its host root.
- `blizzard:unit-test` (`tests/unit/runner/cli`): `runner tick` closes the host root it built on success and on failure.
- `blizzard:unit-test` runs the retargeted runner guards in `tests/guards`, and a planted runner-store construction
  outside the wiring modules fails them.
- `blizzard-context:reference-lint` and `blizzard-context:registry-drift` pass on the host guidance; the wiring exemplar
  passes `blizzard:typecheck` and `blizzard:unit-test`.
- `blizzard:component-test` host readiness and app tests and `blizzard:compat` pass with unchanged HTTP behavior.
