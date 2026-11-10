---
name: reaping-by-pid
code:
  - blizzard/tests/component/runner/harness/opencode/compatibility/test_diagnostic.py::test_timeout_kills_and_reaps_an_opencode_descendant
  - blizzard/tests/component/runner/harness/opencode/compatibility/test_diagnostic.py::test_successful_run_rejects_and_reaps_a_pipe_closing_descendant
---

# Reaping asserted by pid

Replaces the two fixed waits in `test_diagnostic.py`, and with the last of the three waits extends the rule. Delivers
`engineering:event-driven-waits` for that file and `engineering:waits-rule`; builds `spec/waits.md` §Reaping a
descendant and §The rule. Shaped by `decision:test-suite/optimization/reaping-asserted-by-pid`.

## Change

- Each descendant writes its pid to a file as its first act; the marker, the descendant's 0.5 s sleep, and the test's
  0.7 s sleep go.
- Each test polls, bounded at 5 s, until the recorded process is gone.
- `blizzard-context` extends `bzh:service-waits-on-the-system` to every tier by name.

## Acceptance

- Both reaping tests assert the descendant is gone with no fixed sleep.
