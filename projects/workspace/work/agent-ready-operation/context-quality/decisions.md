# Context quality decisions

## Check-source precedence (decision:agent-ready-operation/context-quality/check-source-precedence)

**Decider:** product owner and engineering

**Open.** Choose whether an environment check uses only that checkout's content and declarations or deliberately
combines them with source-checkout rule programs, and how framework documentation checks select version-bound content
rather than an old copied tree; this blocks finalizing `product:quality-content-scope` and source-sensitive
`product:quality-scoped-exemptions`, not changing the context router's version-bound framework promise.

## Contributor accounting (decision:agent-ready-operation/context-quality/contributor-accounting)

**Decider:** product owner and engineering

**Open.** Choose whether coverage counts check programs, individual checks, or contributing owners, naming distinct
counts if necessary, rather than documentation that says scripts while legacy code counts grouped outcomes; this blocks
finalizing accounting in `product:quality-exemption-accountability` and public output compatibility policy.

## Ignore failure severity (decision:agent-ready-operation/context-quality/ignore-failure-severity)

**Decider:** product owner and engineering

**Open.** Choose warning-only handling for unusable ignore declarations or distinguish unreadable authoritative content
as a failing run while keeping typo/staleness warnings, rather than copying the conflicting legacy prose and tests; this
blocks finalizing `product:quality-exemption-accountability`, not the rule that unusable declarations suppress nothing.
