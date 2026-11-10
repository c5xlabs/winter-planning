---
name: silent-daemon-wait
code:
  - blizzard/src/blizzard/runner/cli/daemon.py::daemon_holding
  - blizzard/src/blizzard/runner/cli/daemon.py::Door.uds_client
  - blizzard/tests/unit/cli/test_cli.py::_serve_on
  - blizzard/tests/unit/cli/test_cli.py::test_an_ambiguous_liveness_answer_fails_closed
---

# The silent daemon's wait

Replaces the fixed wait in `tests/unit/cli/test_cli.py`. Delivers `engineering:event-driven-waits` for that file; builds
`spec/waits.md` §The silent daemon. Shaped by `decision:test-suite/optimization/client-timeout-keyword`.

## Change

- `Door.uds_client` and `daemon_holding` take the client timeout as a keyword, defaulting to `LOCAL_CLIENT_TIMEOUT`.
- The silent case passes 0.25 s.
- The fake daemon's hold is an `Event` the fixture sets on teardown.

## Acceptance

- The silent-daemon case finishes in about 0.25 s and still fails closed.
- Every other caller of `daemon_holding` keeps the 5.0 s timeout.
