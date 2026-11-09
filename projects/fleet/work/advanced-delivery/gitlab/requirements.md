# GitLab requirements

Requirements of `epic:advanced-delivery/gitlab`.

## GitLab as a forge kind (product:gitlab-forge)

An operator declares a GitLab server, gitlab.com or their own, as a forge, and a repository on it lands the way a GitHub
repository does. Each of GitLab's words for merge requests, pipelines, branches, and comparisons is mapped into the
seam's, so the policies, the capability check, and the flow never read GitLab's vocabulary.

Acceptance:

- An operator can declare gitlab.com or a self-managed GitLab server as a forge.
- A repository on a GitLab forge lands the way a GitHub repository does.
- A server too old to answer what blizzard relies on is refused when the forge is first used, naming the version it
  found.

## Policies GitLab can honour (product:gitlab-policies)

Merging once checks pass and parking for a person run on GitLab as they do on GitHub. GitLab has no way to move a branch
outside a merge request, so a GitLab repository cannot fast-forward. A project that merges by its own method, a merge
commit, semi-linear history, or a fast-forward, keeps that method.

Acceptance:

- A chunk lands into a GitLab repository by merging once checks pass, and by parking for a person.
- Setting fast-forward on a GitLab repository is refused, naming what the forge lacks.
- The hub records whatever revision the project's own merge method produced.

## Checks read where GitLab runs them (product:gitlab-pipeline-checks)

A project that runs merged-results pipelines, or runs its pipelines only for merge requests, runs its checks on a merge
GitLab builds rather than on the change's head. The hub reads a change's own pipeline, so such a project's checks are
seen and waited on rather than missed.

Acceptance:

- A project running merged-results pipelines has its checks seen and waited on.
- A job that failed and was retried counts once, at its latest outcome.

## A rebase the hub asked for (product:requested-rebase)

GitLab brings a change up to its base by rebasing it, rewriting its head. A rebased head is still the chunk's work when
the hub asked for that rebase itself and what GitLab can say about the result matches what it rebased; any other head is
foreign, as it is today. See `decision:advanced-delivery/gitlab/rebase-head-gate`.

Acceptance:

- A change the hub rebased lands as the chunk's work.
- A rebase nobody asked the hub for, or a person's push after the rebase, is treated as a foreign head.

## Repositories in nested groups (product:nested-groups)

A GitLab project may live several groups deep. A repository on GitLab names its whole namespace.

Acceptance:

- Two projects of the same name in different groups on one server are two repositories, on the runner as on the hub.

## Within GitLab's limits (product:gitlab-rate-limits)

GitLab counts requests by the minute rather than the hour, and may say how much remains only once it refuses. The rate
budget learns from whatever GitLab tells it.

Acceptance:

- A GitLab key slows down rather than fails, exactly as a GitHub key does.

## Landing across kinds of forge (product:cross-forge-landing)

A chunk whose project links a GitHub repository and a GitLab one lands into both, each through its own forge and with
its own policy, in its project's order. Landing across forges is never one atomic act, and the rule
`epic:advanced-delivery/policies` set for landing across repositories holds unchanged.

Acceptance:

- A chunk spanning a GitHub repository and a GitLab one lands both, in its project's order.
- With the GitLab repository parked, the GitHub repository ranked after it waits for the person's merge.
