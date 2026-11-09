---
epic: advanced-deployment
refinement: refined
status: planned
---

# Advanced deployment

A standalone epic; its requirements are in `requirements.md`.

## The problem

A chunk that lands has only finished being written. For an operator whose work has to run somewhere, the night is not
over until the change is live, healthy, and known to be both, and today that last stretch belongs to whatever each
deployment happens to be. One piece of blizzard's own instance is published by CI and pulled onto its host by a timer;
another is rebuilt by hand from a runbook; a third ships through a script. An agent asked to improvise across all of
that is the least trustworthy hand to put on production.

## The outcome

A project has a declared, deterministic way to take landed work the rest of the way to live and proven healthy, with an
agent's judgement called in only where the project asks for it.

Blizzard does not deploy. Nearly every shop already has a pipeline that builds, tests, and ships, and it already holds
the production credentials, the approval gates, and the audit trail. Blizzard uses that pipeline as it stands,
autonomously: it sets it moving or follows it, proves what it produced, and decides what happens next. It never
reimplements CI/CD, and it never needs a cloud credential to do its work.

Kubernetes rollouts, ECS services, GitOps controllers, pull-based updaters, progressive-delivery tools, and a person
with a runbook look nothing alike from the outside. Underneath, each one is a trigger, a run that can be identified, a
run that can be observed, an environment that ends up running some version, a decision to promote or hold, and a way
back, and the epic is built on that shape rather than on any one of those tools. Proof does not depend on mechanism: if
an environment can say which version it is running, and a declared smoke check can say whether it is healthy, blizzard
can prove a deploy without knowing whether a cluster, a container service, a timer, or a human performed it.

Deployment is many chunks to one deploy. Several landings ride one pipeline run, one chunk's deploy is carried by a
later one's, and a rollback undoes every chunk it carried. A chunk's delivery has already ended at landing, so the
deploy is recorded against every chunk it carried and never holds one open.

## Depends on

- `initiative:advanced-delivery`, whose environment flows a deploy runs in, and whose landing policies
  (`epic:advanced-delivery/policies`) a GitOps version bump travels.
