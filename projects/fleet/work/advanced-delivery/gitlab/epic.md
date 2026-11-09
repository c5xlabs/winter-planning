---
epic: advanced-delivery/gitlab
refinement: refined
status: planned
---

# GitLab

Part of `initiative:advanced-delivery`. Its product requirements are in `requirements.md`.

## The problem

A shop whose repositories live on GitLab cannot use blizzard to land anything today, and after
`epic:advanced-delivery/policies` it still cannot. The seam `epic:advanced-delivery/forges` drew speaks in refs,
changes, and checks rather than in GitHub's words, and every policy is written against that seam alone, but the only
forge behind it is GitHub. A seam with one occupant is a promise rather than a proof: nothing yet shows that a policy
written for one forge runs unchanged on another.

## The outcome

GitLab sits behind the seam. A tenant declares a GitLab forge as it declares a GitHub one, a repository on it names a
policy as any repository does, and the hub lands into it through merge requests and pipelines without a policy, a graph,
or the combining step learning which forge it holds. A chunk whose work spans a GitHub repository and a GitLab one lands
into both, each through its own forge, in its project's order. This epic is the evidence that adding a forge means
writing an adapter rather than revisiting the policies.

## Left for later

GitLab's merge trains, as a way to hand a repository's line to its forge, belong to `epic:advanced-delivery/trains`,
which comes after this one and speaks GitHub's merge queue and GitLab's merge train alike.

## What this epic is not

GitLab as a source of work items, rather than a place work lands, is not this initiative's. A forge beyond GitHub and
GitLab waits for a shop that needs one.
