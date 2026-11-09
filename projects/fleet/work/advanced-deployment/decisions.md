# Advanced deployment decisions

## Who declares deploy order (decision:advanced-deployment/deploy-order-owner)

**Open.** Who declares the order repositories deploy in. `initiative:projects` puts a deployment declaration on the
tenant's repository record, while dependency ordering speaks of the dependencies a project declared. A repository two
projects link is deployed once, so dependencies declared per project need a rule for when two projects disagree, and
dependencies declared per repository need a home that is neither project's. It blocks `product:delivery-train`.

## Promotion between environments (decision:advanced-deployment/promotion-chain)

**Open.** Whether promotion between environments is a declared chain inside one train, or a separate train per
environment. It blocks `product:delivery-train`.

## How a flow runs judgement (decision:advanced-deployment/flow-judgement)

**Open.** How a flow runs the judgement it declares, since a flow is not a chunk and has no agent sessions. It could
mint an ordinary chunk on a judgement graph, which must then belong to one project even when the deploy it judges
carried work from two, or it could run a graph's session itself without becoming a chunk. It blocks
`product:declared-judgement`.

## Driver order (decision:advanced-deployment/driver-order)

Drivers are built in roughly this order: follow the pipeline, dispatch the pipeline, probe only, GitOps, then script.
Following comes first because it covers most shops without asking them to change anything. Native platform drivers
(Kubernetes, ECS, Cloud Run) come last if at all, since most organizations reach those platforms through a pipeline and
a native driver means blizzard holding cloud credentials.
