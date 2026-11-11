# Executable contribution decisions

## Shared protocol integration (decision:agent-ready-operation/executable-extensions/protocol-integration)

**Decider:** Platform and Workspace engineering with the product owner

**Open.** Establish which shared Platform owner will define the versioned extension protocol, what Workspace consumes,
and whether legacy version absence is rejected or supported by an explicit external adapter; this blocks finalizing
`product:compatible-contributions` and `engineering:external-participation`, without choosing the shared API locally.

## Execution policy (decision:agent-ready-operation/executable-extensions/execution-policy)

**Decider:** Workspace engineering and the product owner

**Open.** Choose finite execution, output, cancellation-cleanup, and interaction limits together with environment
inheritance and executable trust rules; this blocks finalizing `product:controlled-execution` and
`engineering:invocation-authority`, not the requirement that execution be bounded.

## Hook order and failure policy (decision:agent-ready-operation/executable-extensions/hook-policy)

**Decider:** product owner and lifecycle engineering

**Open.** Choose hook order, behavior after failed structural prerequisites, repeatability, and the overall result for
non-strict hook failures, coordinating the phase boundaries with environment lifecycle and provisioning; this blocks
finalizing `product:lifecycle-contributions` and `engineering:hook-phase-accounting`.

## Conformance authorization (decision:agent-ready-operation/executable-extensions/conformance-policy)

**Decider:** extension-contract engineering and the product owner

**Open.** Choose which conformance claims can be checked without effects and how an author authorizes isolated effectful
checks, including scaffold language and legacy examples; this blocks finalizing `product:safe-conformance` and
`engineering:conformance-isolation`, not the prohibition on implicit live mutation.
