# Tmux provider decisions

## Session reconciliation (decision:running-environments/provider-tmux/session-reconciliation)

**Decider:** product owner and engineering

**Open.** Choose compatibility for existing-session whole-up no-op versus reconciliation, missing-pane partial startup,
layout salvage, dependency closure, and prefix overrides; this blocks finalizing `product:tmux-terminal-services` and
`product:tmux-launch-behavior` without permitting unrelated-pane mutation.

## Log mode support (decision:running-environments/provider-tmux/log-mode-support)

**Decider:** product owner and engineering

**Open.** Choose memory-mode support or refusal, rotation/retention parity, restart capture continuity, pane-follow
fidelity, and workspace-log support under the common log guarantees; this blocks finalizing `product:tmux-observation`,
not exposing unsupported modes or ensuring fair live streams.

## Recovery identity and platforms (decision:running-environments/provider-tmux/recovery-identity)

**Decider:** product owner and engineering

**Open.** Choose recoverable session/process ownership after declaration loss or prefix change and the supported process
inspection/reaping behavior on each operating system; this blocks finalizing `product:tmux-owned-recovery` and its
release acceptance under `decision:supported-systems`, without retaining unsafe suffix-based reaping.
