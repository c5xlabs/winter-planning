# Sentinel agents engineering

Engineering for `epic:sentinel-agents`.

## Brief requirements

- engineering:no-hub-model — The hub schedules sentinel agents and never runs one; every session runs on a volunteered
  runner. Serves `product:runner-volunteers`.
- engineering:turn-grant — A role's exclusivity is a hub-held grant of one turn per role and scope, fenced like a
  worker's claim, so a lost runner's session cannot act after its grant has passed to another. Serves
  `product:exclusive-role` and `product:turn-at-a-time`.
- engineering:turn-commit — The runner drives a sentinel turn by turn rather than letting the harness run free, and
  commits each turn to the hub before asking for the next. Serves `product:every-turn-recorded`.
- engineering:wake-queue — Wakes are queued per role and scope and drained by the one session holding its lease. Serves
  `product:wake-react-stop`.

## Open questions

- **Engineering:** How much of the worker's lease, heartbeat, and fencing machinery a turn grant reuses, given that a
  sentinel agent holds no chunk.
- **Engineering:** Whether the turn driver the OpenCode 2 binding already uses generalizes to driving every harness one
  turn at a time, or whether some harness cannot be driven that way.
- **Engineering:** Whether an interval wake rides `epic:cadence`'s scheduling or needs its own.
