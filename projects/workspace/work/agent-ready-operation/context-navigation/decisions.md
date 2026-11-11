# Context navigation decisions

## Safety hook delivery (decision:agent-ready-operation/context-navigation/safety-hook)

**Decider:** product owner and harness engineering

**Open.** Choose the minimum eager text and optional session-start delivery that preserve always-on safety across
supported harnesses without eagerly loading all framework context; this blocks finalizing
`product:eager-workspace-safety` and the corresponding projection contract.

## Context addressing (decision:agent-ready-operation/context-navigation/topic-addressing)

**Decider:** product owner and engineering

**Open.** Choose stable topic/source addresses and how existing extension path notation resolves through the common
router; this blocks finalizing `product:unified-context-sources`, not the requirement for progressive disclosure.
