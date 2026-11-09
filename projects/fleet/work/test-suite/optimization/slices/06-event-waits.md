---
name: event-waits
---

# Waits on events

Replaces the three fixed waits, in one change per file, the last landing the rule's extension. Delivers
`engineering:event-driven-waits` and `engineering:waits-rule`; builds `spec/waits.md`.

## Change

- `tests/unit/cli/test_cli.py`: the client timeout passed as a keyword, and the fake daemon's hold an `Event`.
- `tests/component/runner/harness/opencode/compatibility/test_diagnostic.py`: the descendant's reaping asserted by pid.
- `blizzard-context` extends `bzh:service-waits-on-the-system` to every tier by name, with the last of the three.

## Acceptance

- The silent-daemon case finishes in about a quarter of a second and still fails closed.
- Both reaping tests assert the descendant is gone with no fixed sleep.
