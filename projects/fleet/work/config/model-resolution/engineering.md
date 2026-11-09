# Model resolution engineering

Derives from `epic:config/model-resolution`.

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
