# Riding out provider outages engineering

Engineering for `epic:resilience`.

## Brief requirements

- engineering:held-lease — Work waiting out an outage keeps its lease. Serves `product:hold-place`.
- engineering:waiting-not-dead — A worker holding a lease through an outage is never treated as dead by heartbeating or
  fencing. Serves `product:hold-place`.
- engineering:shared-release — Handing held work to another runner reuses the release `epic:throttling` gives a runner
  whose gate parks a chunk mid-graph, built once for both. Serves `product:fallback-first`.

## Open questions

- **Engineering:** How an outage is recognized through each harness, whose failures arrive as exit codes and stderr
  rather than as a status an API would hand over.
- **Engineering:** What a held lease means to heartbeating and fencing while the worker is deliberately doing nothing.
