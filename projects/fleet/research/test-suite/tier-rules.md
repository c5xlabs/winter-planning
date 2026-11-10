# Test suite tier rules

The detail behind the test suite's rules for what a test may touch and where it lives.

## The crash sweep

Behind `decision:test-suite/crash-sweep-stays`. The crash sweep asks whether a process killed at an exact line leaves
its store consistent, and both halves of that question belong to this implementation: its crash points mark boundaries
in this code, and most of its invariants describe how this code keeps its books. A rewrite would bring different
boundaries and could keep every promise a person sees while breaking every one of those invariants. Whatever the sweep
borrows from a file the fleet tier deletes moves to a Python support module first.

## Public surfaces

Behind `decision:test-suite/public-surfaces`.

- A test that reads or writes the store proves a state the hub can never reach on its own.
- `blizzard-mock-data` serves agents and people testing by hand, not tests.
- Where a test needs state no public surface can produce, the order of preference is the real flow, then a surface an
  operator or an integration could use as well, shaped for the general case rather than for one test and secured like
  every other, and only when nothing reasonable serves, a test-only route behind a configuration flag.
- A `fresh-hub` test whose subject is the store touches the store's files only before the daemon starts or after it
  stops.

## Moving a test

Behind `decision:test-suite/move-once`.

- Any helper another tier imports from the deleted file moves to a support module first.
- The port keeps the original's function name as its title, so the harness citations that name it move mechanically.

## Package layout

Behind `decision:test-suite/package-layout`.

- A service test brings up one real thing with mocks around it, in three shapes: the hub alone in `tests/service/hub/`,
  a runner host alone in `tests/service/runner/`, and both together in `tests/service/fleet/`.
- A UI test, in `tests/ui/`, drives the built board in a browser against a real hub, with the same mocks behind it.
- The word end-to-end is not used until blizzard can stand up a real work source and a real GitHub repository under
  test.
- The package shares `tests/` with the Python suites only until they have moved.
