# Agent identity and capabilities engineering

Engineering for `epic:agent-identity`.

## Brief requirements

- engineering:runner-token-unchanged — The runner's own credential keeps exactly the reach it has today; an
  agent-operator's credential is a separate, short-lived one the hub issues per work. Serves `product:work-identity` and
  `product:scoped-credential`.
- engineering:checked-at-the-hub — Altitude and the queuer's bound are enforced by the hub on every act the credential
  makes, never by the runner or the agent. Serves `product:altitudes` and `product:bounded-by-queuer`.

## Open questions

- **Engineering:** How the credential reaches the agent's session without passing through a place a builder's session
  could read it, on a runner that may run both.
- **Engineering:** Whether the agent acts through the blizzard CLI with the issued credential, the same surface a person
  uses, or through a narrower one.
