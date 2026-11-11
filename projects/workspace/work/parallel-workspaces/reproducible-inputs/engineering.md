# Reproducible input engineering requirements

For `epic:parallel-workspaces/reproducible-inputs`.

## Resolution record integrity across writers (engineering:resolution-write-integrity)

Concurrent processes updating different inputs cannot lose each other's records, and interrupted checkout/record changes
remain detectable rather than presenting a false completed resolution. Malformed records are never treated as absent.
Serves `product:recorded-input-materialization` and `product:update-standalone-inputs`.

## Preserve temporary work on every exit path (engineering:repin-preservation-lifetime)

Temporary preservation created for an update remains discoverable after resolution failure, checkout failure,
interruption, or restoration conflict; cleanup never drops the only remaining copy. Serves
`product:update-standalone-inputs`.

## Content identity cannot depend on stat freshness (engineering:fingerprint-observation)

Fingerprinting detects same-size edits in one timestamp tick, accounts for staged-versus-working state, and preserves
the real Git index byte-for-byte. Concurrent edits cannot produce an unexplained claim of a coherent snapshot; any
allowed object-store-only writes are disclosed under the identity decision. Serves `product:workspace-content-identity`.
