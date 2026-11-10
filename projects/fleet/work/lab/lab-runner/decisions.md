# Lab runner decisions

## Arm definition reference (decision:lab/lab-runner/arm-definition)

**Open.** Whether the arm reference the hub stores names a definition the lab workspace already holds, which makes arms
something an engineer prepares, or a workspace repository and the commits to pin it to, which the runner materializes on
first use so an experiment can ask for an arm on its own; it blocks `product:arm-workspaces`.
