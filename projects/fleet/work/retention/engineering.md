# Retention engineering

Engineering for `epic:retention`.

## Brief requirements

- engineering:expiry-sweep — Expiry runs as one of the hub's sweeps on its own timer and cost floor, as fact egress's
  export does, never as runner work (`decision:retention/expiry-as-sweep`). Serves `product:retention-policy`.
