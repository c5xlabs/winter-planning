# Harness projection decisions

## Artifact ownership (decision:agent-ready-operation/harness-projection/artifact-ownership)

**Decider:** product owner and Workspace engineering

**Open.** Choose how generated ownership is established and recovered, whether edited owned artifacts are retained,
backed up, or replaced only with authorization, and how legacy/invalid ownership is adopted; this blocks finalizing
`product:projection-ownership` and `engineering:projection-authority`.

## Last good output (decision:agent-ready-operation/harness-projection/last-good-policy)

**Decider:** product owner and harness engineering

**Open.** Choose retention versus withdrawal for invalid agents, incomplete mappings, unavailable source checkouts, and
refused permission projection, including per-harness versus all-harness installation; this blocks finalizing
`product:projection-recovery`, not the distinction between render failure and source removal.

## Concurrent writes and recovery (decision:agent-ready-operation/harness-projection/write-recovery)

**Decider:** Workspace engineering

**Open.** Choose cross-process exclusion/refusal, recoverable copy replacement, and ownership commit boundaries without
claiming an atomic transaction across all harnesses; this blocks finalizing `engineering:projection-failure-accounting`
and the concurrent-reconciliation acceptance of `product:projection-recovery`.

## Supported harness versions (decision:agent-ready-operation/harness-projection/harness-compatibility)

**Decider:** product owner and harness engineering

**Open.** Choose tested Claude Code, Codex, and OpenCode version baselines, artifact-discovery compatibility, and how
unsupported native fields are diagnosed; this blocks release acceptance for `product:skill-discovery` and
`product:agent-discovery`, not their three-harness scope.
