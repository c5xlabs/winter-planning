# Lab runner decisions

## Arm definition reference (decision:lab/lab-runner/arm-definition)

**Open.** How a lab runner learns an arm's definition. The reference the hub stores might name a definition the lab
workspace already holds, or a workspace repository and the commits to pin it to, which the runner materializes on first
use. The first makes arms something an engineer prepares; the second lets an experiment ask for one on its own. It
blocks `product:arm-workspaces`.
