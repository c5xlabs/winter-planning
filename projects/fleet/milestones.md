# Milestones

Where the fleet is headed, as destinations in the operator's work, in the order they are meant to be reached. Each
milestone names the epics needed to reach it; each epic's own status is the truth, so this is a plan, not a status
board. Once a milestone is reached, it moves to the [delivered ledger](./delivered.md).

## Hardening (milestone:hardening)

Decide for themselves how the fleet behaves: every operational constant theirs to set, when and at what rate each runner
takes work theirs to declare, and every upgrade taken when they choose and told beforehand what it asks of them.

| Epic                                     | Status      |
| ---------------------------------------- | ----------- |
| `epic:runner-host`                       | delivered   |
| `epic:release-engineering/first-release` | delivered   |
| `epic:release-engineering/compat-floor`  | delivered   |
| `epic:release-engineering/notes`         | delivered   |
| `epic:release-engineering/release-lines` | delivered   |
| `epic:release-engineering/dark-launch`   | delivered   |
| `epic:release-engineering/supply-chain`  | delivered   |
| `epic:storybook`                         | delivered   |
| `epic:throttling`                        | delivered   |
| `epic:test-suite/foundation`             | delivered   |
| `epic:test-suite/hub-tier`               | delivered   |
| `epic:test-suite/runner-tier`            | delivered   |
| `epic:test-suite/fleet-tier`             | delivered   |
| `epic:test-suite/journey`                | delivered   |
| `epic:architectural-sweep`               | delivered   |
| `epic:strict-types`                      | delivered   |
| `epic:strict-mock`                       | delivered   |
| `epic:test-suite/controlled-clock`       | delivered   |
| `epic:config/model-resolution`           | planned     |
| `epic:config/sweep`                      | planned     |
| `epic:config/runtime`                    | in-progress |
| `epic:ui-toolkit`                        | planned     |
| `epic:cli-context`                       | planned     |
| `epic:test-suite/optimization`           | planned     |
| `epic:adopt-di`                          | planned     |
| `epic:transactions`                      | in-progress |
| `epic:advanced-forge-auth`               | planned     |

## Autonomy (milestone:autonomy)

Leave the fleet to run itself: it keeps its own schedule, takes up work the moment something happens, reaches first for
what is most urgent, hands each piece to the runner and the settings suited to it, and clears the road for a change too
large to share it — without the operator choreographing any of it.

| Epic                         | Status      |
| ---------------------------- | ----------- |
| `epic:cadence`               | delivered   |
| `epic:signals`               | planned     |
| `epic:urgency`               | in-progress |
| `epic:tagging`               | planned     |
| `epic:worker-profiles`       | planned     |
| `epic:chokepoint/chokepoint` | planned     |
| `epic:chokepoint/epic-dwf`   | planned     |

## Foundational seams (milestone:foundational-seams)

Run blizzard on the tools they already use, not only the ones it was first built on: each seam the fleet was drawn along
— the coding harness first — opened from the one tool it shipped with to the range of tools its users actually work in.

| Epic                  | Status  |
| --------------------- | ------- |
| `epic:adapters/codex` | planned |

## Human in the loop (milestone:human-in-the-loop)

Stop being the wire between the fleet and everything it needs: it reaches them wherever they are when a decision is
genuinely theirs, settles CI's verdict itself when it is not, and takes a word of direction mid-flight without the
session being taken over.

| Epic               | Status  |
| ------------------ | ------- |
| `epic:chat`        | planned |
| `epic:ci-feedback` | planned |
| `epic:steering`    | planned |

## Mobile (milestone:mobile)

Carry the fleet in a pocket: watch the night, answer a question, and unblock a chunk from a phone, through notifications
that arrive the way the phone's own do.

| Epic           | Status  |
| -------------- | ------- |
| `epic:pwa`     | planned |
| `epic:android` | planned |
| `epic:ios`     | planned |

## Future state

Ideas for where the fleet goes next, not yet needed by any milestone.

| Epic                                      | Status  |
| ----------------------------------------- | ------- |
| `epic:preview`                            | planned |
| `epic:provider-kit`                       | planned |
| `epic:terraform-provider`                 | planned |
| `epic:tenant-telemetry`                   | planned |
| `epic:demo`                               | planned |
| `epic:code-harness-window/agent-observer` | planned |
| `epic:code-harness-window/swarm-view`     | planned |
| `epic:lab/bench`                          | planned |
| `epic:lab/experiment`                     | planned |
| `epic:lab/lab-runner`                     | planned |
| `epic:lab/specimen`                       | planned |
| `epic:lab/verdict`                        | planned |
| `epic:lab/lab-board`                      | planned |
| `epic:lab/seeded-defects`                 | planned |
| `epic:lab/evidence`                       | planned |
| `epic:advanced-delivery/gitlab`           | planned |
| `epic:advanced-delivery/trains`           | planned |
| `epic:advanced-deployment`                | planned |
| `epic:retention`                          | planned |
| `epic:ui-redesign`                        | planned |
| `epic:resilience/provider-outages`        | planned |
| `epic:documentation`                      | planned |
