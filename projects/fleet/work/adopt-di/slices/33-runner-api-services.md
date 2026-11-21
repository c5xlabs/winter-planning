---
name: runner-api-services
depends-on:
  - runner-loop
  - hosted-runners
  - host-records
code:
  - blizzard/src/blizzard/runner/app.py::_wire_runner_clients
  - blizzard/src/blizzard/runner/app.py::create_app
  - blizzard/src/blizzard/runner/app.py::runner_seams
  - blizzard/src/blizzard/runner/app.py::create_app_for_export
  - blizzard/src/blizzard/runner/api/wiring.py::bind_runner
  - blizzard/src/blizzard/runner/api/wiring.py::HostRunnerRegistry
  - blizzard/src/blizzard/runner/api/wiring.py::ServedRunner
  - blizzard/src/blizzard/runner/api/invocation_usage.py::router
---

# Runner API resolution mechanism

Builds the runner API edge in spec [Entrypoint resolution](../spec/entrypoints.md#interfaces) and the service
registrations it resolves, with one pilot route module. Delivers `engineering:narrow-api-edge`'s route portion,
`engineering:every-route-resolves` and `engineering:born-in-container` under `decision:adopt-di/declared-cutover`. The
route families migrate in `runner-api-host`, `runner-api-leases`, `runner-api-control`, `runner-api-settings` and
`runner-api-telemetry`.

## Change

- Register the per-runner services `_wire_runner_clients` builds today, its two authenticated hub clients, and
  `RunnerSettingsAuthoring` at runner lifetime. `RunnerStepper`, which `_wire_runner_clients` also builds, is
  `runner-loop`'s.
- Register `create_app`'s app-level services at host lifetime: `SubscriptionFloorClearService`, `HttpHubAuthMode`,
  `JwksCache`, the `IJtiCache` from `host_stores.jti_cache`, `TrustedProxies`, the session secret, the default hub
  client, and the values behind the `hub_origin` (`HubOrigin`), `federation` (`FederationSettings`) and
  `hub_retry_clock` slots. Give each `runner_seams` default an explicit factory.
- `HostRunnerRegistry` entries hold each served runner's child container beside `ServedRunner.seams`; `bind_runner`
  attaches it to the request, so a route can declare its services while unconverted routes still read the seams.
- Add `ServedRunners` from the same `current()` read for host routes that report per runner, skipping a runner whose
  child is closing.
- Add `host_request`, the router dependency that attaches the host container for host routes.
- Register each service an app can be built without through `Registry.absent` with the label its `RunnerWiring` accessor
  gives today, and add the runner app's app-level handler that answers `ServiceAbsent` with that label's `503`; a route
  takes the service as plain `FromDishka[T]`. Register each degrading `maybe_` read's service through
  `Registry.optional`, so a route takes `FromDishka[T | None]`. `create_app_for_export` and partial test apps validate
  whole.
- Add the runner app's app-level exception handler that answers a `ScopeClosed(RUNNER, runner_id)` with the `404` a
  withdrawn runner answers (`decision:adopt-di/drained-runner-request`); any other `ScopeClosed` is not caught by it.
- Move the middleware readers of the served runners onto the registry and child context values, per
  [Entrypoint resolution](../spec/entrypoints.md#interfaces): `_served_identity` in the identity-stamping middleware,
  `_FirstServedIdentity`, and `_gate_web_surface`'s `require_human_session`, which resolves only host services the root
  builds eagerly. None constructs lazily, and each skips a runner whose child is closing.
- Asynchronous runner handlers take through `FromDishka` only host-lifetime services the root builds eagerly; anything
  else comes from a synchronous dependency.
- Migrate `runner/api/invocation_usage.py`, a runner-scoped route reading one store and importing no other route module,
  from `RunnerWiring` accessors to declared services as the pilot.
- Add a unit-tier walker that resolves every runner-app route's declared services at its actual scope, runner or host,
  over the full and the export app.

## Acceptance

- `blizzard:component-test`: on the invocation-usage route, each refusal `RunnerWiring` gives today (`503` with its
  label, `404` unknown runner, `503` halted runner) has the same status and payload; the export app builds and
  validates, and an absent service answers its label's `503`.
- `blizzard:component-test`: a request whose runner child closes during the identity-stamping middleware is stamped from
  no closed child and does not answer `500`; the web-surface gate behaves as today.
- `blizzard:component-test`: a request whose runner is drained while it resolves answers `404`; a `ServedRunners`
  iteration over two runners, one draining, yields the other.
- `blizzard:unit-test`: the every-route walker passes, and a planted route declaring a service its scope lacks fails it.
- `blizzard:typecheck`, `blizzard:compat` and `blizzard:sse-contract` pass with unchanged wire behavior.
