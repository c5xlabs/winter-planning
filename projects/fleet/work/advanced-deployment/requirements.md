# Advanced deployment requirements

The requirements of `epic:advanced-deployment`.

## A deployment declaration per environment (product:deployment-declaration)

Each repository declares, per environment, how it is deployed: the environment, the environments it must follow, the
driver that moves it, how to read its live version, the smoke checks that prove it healthy, the rollback procedure, and
how safe that rollback is. The declaration belongs to the repository, which the tenant holds and projects link; who
declares the order repositories deploy in is `decision:advanced-deployment/deploy-order-owner`.

Acceptance:

- An operator declares each of these for every environment a repository deploys to.

## Drivers that say what they can do (product:driver-capabilities)

Each driver starts or finds a run, reports its state (queued, running, awaiting approval, succeeded, failed), hands over
the evidence a judging agent reads, and optionally approves, promotes, or rolls back natively. A declaration that asks
for a capability its driver lacks is refused when it is declared, not discovered mid-deploy. The order drivers arrive in
is `decision:advanced-deployment/driver-order`.

Acceptance:

- A declaration asking a driver for a capability it lacks is refused at declaration, naming the capability.
- The operator sees each run's state as one of queued, running, awaiting approval, succeeded, or failed.

## Follow the pipeline (product:follow-pipeline)

When the landing already started the pipeline, blizzard finds the run for the landed commit, waits it out, then proves
the environment. This covers most shops without asking them to change anything, GitHub Actions and GitLab CI first.

Acceptance:

- A landing on a repository deployed by GitHub Actions or GitLab CI is followed to a proven deploy with no change to the
  pipeline.

## Dispatch the pipeline (product:dispatch-pipeline)

Blizzard starts a named pipeline with inputs, for promotions and for rollback pipelines.

Acceptance:

- A promotion or rollback starts the named pipeline with its declared inputs.

## Probe only (product:probe-only)

For a pull-based updater, and any process blizzard cannot see into, blizzard triggers nothing and waits for the
environment to report the version.

Acceptance:

- A probe-only environment is proven once it reports the landed version, with nothing triggered.

## GitOps (product:gitops-deploy)

Deploying is a version bump committed to a configuration repository, then a wait for the controller to report synced and
healthy. That commit is itself a landing, so it travels the landing policy `epic:advanced-delivery/policies` builds.

Acceptance:

- The version bump lands under the configuration repository's landing policy.
- The deploy is proven once the controller reports synced and healthy and the environment reports the version.

## Script (product:script-deploy)

A declared deterministic command performs a deploy that has no pipeline, with an explicit statement of where it runs:
some deploys can only be performed on one particular machine, and the hub never reaches into a runner's.

Acceptance:

- A script deploy runs on the machine its declaration names.

## Proven by version and checks (product:deploy-proof)

A deploy is done when the environment reports a version that contains the landed commit and its smoke checks pass,
whichever driver ran. A pipeline that succeeded has not proven a deploy, since it can finish long before its output runs
anywhere. The target is the commit the branch actually moved to: a later commit that contains it proves it deployed, and
a run for a later commit may be the run that carried it.

Acceptance:

- A succeeded pipeline whose environment has not reported the version is not shown as deployed.
- An environment reporting a later commit that contains the landed one proves the landed one deployed.
- Landings batched into one run, or a superseded run, still reach proven.

## Delivery trains (product:delivery-train)

Once a chunk lands, the train works out which repositories changed, orders their deployments by the dependencies
declared for them, and moves each only after the one before it is proven. Whether promotion between environments is a
chain inside one train is `decision:advanced-deployment/promotion-chain`.

Acceptance:

- A repository whose deploy depends on another's starts only after that one is proven.

## Judgement where declared (product:declared-judgement)

A step a script can decide is never handed to a model. A deploy that needs more than a deterministic verdict runs a
graph on a chosen model: one agent reads the evidence and decides healthy, broken, or due for rollback; the declared
rollback runs as a deterministic step; a second agent confirms the rollback took. An agent deploying by hand remains
available, as the fallback rather than the method. How a flow runs that graph is
`decision:advanced-deployment/flow-judgement`.

Acceptance:

- No model is called on a deploy whose declaration asks for none.
- A rollback a judging agent calls for is run by the declared procedure and confirmed by a second agent.

## Rollback as safe as declared (product:rollback-safety)

A deploy that ran a migration cannot always be undone by swapping the version back, as blizzard's own hub updater
learned. Each declaration states whether rollback is safe, needs a human, or is forbidden in favor of rolling forward,
so no agent improvises the answer.

Acceptance:

- A rollback declared as needing a human waits for one.
- A rollback declared as forbidden is never run; the deploy rolls forward.

## Approval gates surfaced, never bypassed (product:approval-gates)

A pipeline's own required reviewers and manual jobs arrive in blizzard as asks for the person who holds the decision.

Acceptance:

- A pipeline waiting on a required reviewer or manual job shows as an ask to that person.
- Blizzard never approves past a gate on its own.

## One deploy per environment at a time (product:environment-exclusivity)

Two deploys to one environment never overlap, while different environments proceed side by side. When several landings
queue behind a deploy, the newest version goes, and the ones it contains are recorded as carried rather than deployed
one by one.

Acceptance:

- A second deploy to a busy environment waits; one to another environment does not.
- Landings superseded in the queue show as carried by the deploy that contained them.

## Every wait ends somewhere named (product:named-wait-outcomes)

Queueing, running, awaiting approval, and converging each keep their own clock, and none falls through to success. A run
that never appears, a version that never arrives, and a check that cannot run each end in an outcome the project
authored. Where a failure happened decides what it means: a pipeline that failed before its deploy changed nothing and
is fixed forward, while one that failed during it leaves the environment in doubt and goes to judgement.

Acceptance:

- A run that never appears, a version that never arrives, and a check that cannot run each reach their authored outcome
  when their own timeout passes.
- A failure before the deploy step is reported as nothing changed; one during it goes to judgement.

## Deploys recorded on every chunk carried (product:deploy-on-chunks)

Every chunk a deploy carried is told it was deployed or rolled back, and the deploy never holds a chunk's delivery open.

Acceptance:

- Each chunk a deploy carried shows that deploy's outcome.
- A rollback shows on every chunk it undid.
