# Advanced deployment decisions

## Who declares deploy order (decision:advanced-deployment/deploy-order-owner)

**Open.** Who declares the order repositories deploy in, since `initiative:projects` puts the deployment declaration on
the tenant's repository record while dependency ordering speaks of dependencies a project declared. A repository two
projects link deploys once, so per-project dependencies need a rule for when the projects disagree and per-repository
ones need a home that is neither project's; it blocks `product:delivery-train`.

## Promotion between environments (decision:advanced-deployment/promotion-chain)

**Open.** Whether promotion between environments is a declared chain inside one train, or a separate train per
environment. It blocks `product:delivery-train`.

## How a flow runs judgement (decision:advanced-deployment/flow-judgement)

**Open.** How a flow, which is not a chunk and has no agent sessions, runs the judgement it declares: by minting an
ordinary chunk on a judgement graph, which must belong to one project even when the deploy it judges carried work from
two, or by running a graph's session itself without becoming a chunk; it blocks `product:declared-judgement`.

## Driver order (decision:advanced-deployment/driver-order)

Drivers are built in roughly the order follow the pipeline, dispatch the pipeline, probe only, GitOps, then script,
because following covers most shops without asking them to change anything. Native platform drivers (Kubernetes, ECS,
Cloud Run) come last if at all, since most organizations reach those platforms through a pipeline and a native driver
means blizzard holding cloud credentials.
