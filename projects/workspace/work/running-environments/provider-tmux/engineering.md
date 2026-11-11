# Tmux provider engineering requirements

For `epic:running-environments/provider-tmux`.

## What already holds

`engineering:provider-contract`, `engineering:external-code-boundary`, and `decision:rust-replacement` apply to the
provider and its maintained helpers. `engineering:cli-runtime-baseline` owns common CLI targets;
`decision:supported-systems` owns tmux support and external prerequisites on those targets. Built-in execution placement
is not fixed here.

## Native provider completeness (engineering:tmux-native-workflow)

The complete maintained tmux workflow implements common provider conformance without delegating control, capture,
manifest handling, convenience commands, or recovery to Python. Native behavior retains operator-authored shell hooks as
external execution rather than translating their contents. Serves `product:tmux-terminal-services`.

## Launch and probe consistency (engineering:tmux-environment-boundary)

Selected launch/restart preflight and health evaluation share explicit scope/source/mapping semantics while observation
keeps lookup execution disabled. Server environment sanitization and pane runtime resolution prevent one scope's
inherited state becoming another's baseline; hook environments do not accidentally receive service mappings. Serves
`product:tmux-launch-behavior` and `product:tmux-observation`.

## Ownership-bounded termination (engineering:tmux-process-ownership)

Recovery never uses an environment-name suffix alone as authority on a shared server; signaling and escalation are
bounded to validated owned resources and preserve termination failures. Process discovery, uptime, and reaping state
their supported platform semantics rather than assuming identical external-tool flags across operating systems. Serves
`product:tmux-owned-recovery` and `product:tmux-observation`.

## Capture lifecycle (engineering:tmux-capture-boundaries)

Capture and retention operate only on owned log artifacts, preserve partial-line boundaries during follow, and report
rotation/truncation limitations. Retry, dependency waits, and follow cancellation terminate invocation-owned helpers
without ending unrelated workloads. Serves `product:tmux-observation` and `product:tmux-launch-behavior`.
