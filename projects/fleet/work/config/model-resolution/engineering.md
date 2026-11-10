# Model resolution engineering

Derives from `epic:config/model-resolution`.

## What already holds

- The hub merge: `EffectiveSession.of(chunk, graph, node)` combines a declared session with the chunk's defaults field
  by field and stamps `session_model`, `session_effort`, and `session_harnesses` onto the node envelope, with no record
  of which layer supplied a field.
- Harness selection: `HarnessSelector.select(node)` walks the acceptable set in order and returns the first member that
  can serve the session, with the members it skipped.
- Adapter resolution: each binding's `resolve_model`, `resolve_model_strict`, and `resolve_effort`, over the shared
  left-to-right skeleton in `harness_shared`, turn authored strings into what reaches the harness process.
- A resume never re-resolves: it replays the `resolved_model` and `resolved_effort` recorded on the lease that minted
  the session, so an alias edited after a pool was minted changes the next fresh session.

## Brief requirements

- engineering:read-only-report — The command writes nothing to the configuration, the runner store, or the hub, and
  opens no write path to the store. Serves `epic:config/model-resolution`.

## One resolution, traced (engineering:traced-resolution)

The report is built from the same hub merge, harness selection, and adapter resolution the spawn path runs, each in a
traced form that returns the value with the layer that supplied it, and never from a second implementation of their
rules. The existing entry points are projections of the traced forms, so the spawn path and the report cannot disagree.
Serves `product:terminal-and-json` and `product:cell-provenance`.

Acceptance:

- For every configuration fixture and authored session in the suite, the traced value equals what
  `SessionResolver.session_stamps` returns for a fresh mint, over fixtures that exercise every source in the provenance
  vocabulary.
- A merge test per field shows the source change from `graph` to `chunk` exactly when the declaration's value is empty
  or null, alongside the existing test pinning the precedence itself.

## The hub's merge, not a copy (engineering:hub-merge-read)

The runner takes each session's merged fields, with each field's source, from the hub that already performs the merge,
instead of re-deriving them from graph files that may not match what was minted. Serves `product:session-pools`.

Acceptance:

- A runner token without a lease is answered by the sessions routes.
- A request without a runner token is refused.
- An unknown graph or chunk returns `404` with the name that did not resolve.
