# Reproducible input decisions

## Reference resolution and lock meaning (decision:parallel-workspaces/reproducible-inputs/resolution-and-locks)

**Decider:** product owner and engineering

**Open.** Choose branch/tag ambiguity, moving-branch versus recorded-commit precedence, stale-record handling, and
behavior when a recorded object is unavailable; this blocks finalizing `product:standalone-reference-policy` and
`product:recorded-input-materialization`, with legacy-format support owned by `decision:configuration-transition`.

## Update preservation (decision:parallel-workspaces/reproducible-inputs/update-preservation)

**Decider:** product owner and engineering

**Open.** Choose whether deliberate repinning may replace local-only committed history, how discard approval is scoped,
and restoration-conflict recovery; this blocks finalizing `product:update-standalone-inputs`, not work protection.

## Content identity boundary (decision:parallel-workspaces/reproducible-inputs/fingerprint-boundary)

**Decider:** product owner and engineering

**Open.** Choose whether component version, local configuration, submodules, and checkout filters belong in definition
identity and what concurrent-edit consistency or object-store writes are acceptable; this blocks finalizing
`product:workspace-content-identity` without selecting a digest algorithm or freezing a legacy serialization.
