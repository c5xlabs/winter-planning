# Test tier framework

The case for Playwright Test as the tier's framework, behind `decision:test-suite/playwright`.

## What Playwright Test offers

- Its fixtures are scoped, injected by name, built only when a test asks for them, and composed from one another, so a
  test that names a tenant gets the hub beneath it without arranging either.
- Its projects are the natural shape for hub configuration profiles.
- Writing the tier in another language enforces the black-box line by construction, since a TypeScript test cannot
  import `blizzard.*`.
- Playwright runs TypeScript without a build, and the gate checks its types.

## Alternatives weighed, not spiked

- **pytest in a package sealed from `blizzard`** has the same fixture model and would keep most test bodies, but leaves
  the boundary to packaging rather than language.
- **Go**, with the standard library or Ginkgo, orchestrates processes and reads streams well but has no fixture model of
  comparable depth.
- **Rust**'s nextest runs each test in its own process, which fights a hub shared across tests.
- **Hurl and Venom** cannot hold an event stream open or own a daemon's lifecycle.

## The spike

A spike against a real hub retired the hub side's risks and found one ceiling, a single hub's one core. The foundation
carries what it proved and what it asks of the product. The spike never reached the runner, so the runner tier opens
with a spike of its own.
