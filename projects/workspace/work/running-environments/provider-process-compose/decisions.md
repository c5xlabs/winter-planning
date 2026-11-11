# Process-compose provider decisions

## Recovery platforms (decision:running-environments/provider-process-compose/recovery-platforms)

**Decider:** product owner and engineering

**Open.** Choose supported operating systems and stable process ownership evidence for stale recovery, including daemon
identity reuse and control endpoints outliving their root; this blocks finalizing `product:process-compose-recovery`
under `decision:supported-systems` without treating the current Linux session scan as portable or sufficient proof.

## Supervisor compatibility (decision:running-environments/provider-process-compose/supervisor-compatibility)

**Decider:** product owner and engineering

**Open.** Choose the supported supervisor version baseline from the observed 1.122.0 behavior, including update effects,
readiness failure thresholds versus caller deadlines, startup/restart precedence, and log truncation; this blocks
finalizing `product:process-compose-feature-limits`, `product:process-compose-launch`, and
`product:process-compose-observation` without prescribing a generated configuration schema.

## Portable declaration behavior (decision:running-environments/provider-process-compose/portable-declarations)

**Decider:** product owner and engineering

**Open.** Choose environment/probe parity with tmux and support or explicit refusal for interactive commands, log/uptime
probes, cross-provider dependencies, and image-only contributions; this blocks finalizing
`product:process-compose-feature-limits` and `product:process-compose-launch`, not truthful capability reporting.
