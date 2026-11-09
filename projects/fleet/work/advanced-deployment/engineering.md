# Advanced deployment engineering

Engineering requirements for `epic:advanced-deployment`.

## What already holds

Most of the engine exists:

- Hub-executed command nodes start and follow runs.
- The pending outcome, with its poll interval and timeout, is the wait a pipeline needs.
- Authored choices route healthy, broken, and rollback.
- Markers make a retried deploy skip what is already live.
- `epic:advanced-delivery/flows` settles that a flow is not a chunk, so it has none of the agent sessions a chunk is
  given.

New in this epic: the declaration, the drivers and probes, per-environment exclusivity in place of the one slot a tenant
shares, dependency ordering, evidence gathering, and scripts that execute on a runner.

## Brief requirements

- engineering:pipeline-permission-only — Blizzard holds permission to start a named pipeline and read its status, and no
  production or cloud credential. Serves `epic:advanced-deployment`.

## Environment flows (engineering:environment-flow)

A deploy is a step holding a named resource, its environment, in a flow over that environment rather than in a chunk's
graph. An environment flow consumes what a branch flow landed and reports deployed or rolled back to every chunk it
carried. Serves `product:deploy-on-chunks`, `product:environment-exclusivity`.

## Per-environment lock (engineering:environment-lock)

Each environment is its own exclusive resource, replacing the one deploy slot a tenant shares, so deploys to one
environment serialize and deploys to different environments run concurrently. Serves `product:environment-exclusivity`.

## Runner-executed scripts (engineering:runner-scripts)

A script deploy executes on a runner; the hub never reaches into a runner's machine. Serves `product:script-deploy`.

## Open questions

- How a live version is tied back to a commit when the environment reports something else: a CI run number, an image
  digest, a release tag.
- How a run is identified with certainty when one commit starts several pipelines, or a pipeline is re-run.
- Where evidence beyond pipeline logs comes from, such as metrics and log queries against the operator's observability
  stack, and whether that is a plug-in of its own.
- How a script that must run on a particular machine is routed to a runner that holds the right credentials, without the
  hub ever reaching into that machine.
