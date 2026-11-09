# GitLab specifications

The technical contracts behind `epic:advanced-delivery/gitlab`. Product intent begins at the [epic plan](../epic.md);
implementation enters at the concern below. The forge record, the seam's roles, its vocabulary, and its retry rules are
owned by the [forges epic](../../../../delivered/advanced-delivery/forges/spec/index.md); the policies, the seam
operations they added, the order a project lands its repositories in, the combining step, and the rate budget by the
[policies epic](../../../../delivered/advanced-delivery/policies/spec/index.md). This spec adds a second adapter behind
that seam and changes no policy.

This spec assumes the [actions](../../../../delivered/advanced-delivery/actions/spec/index.md),
[forges](../../../../delivered/advanced-delivery/forges/spec/index.md),
[flows](../../../../delivered/advanced-delivery/flows/spec/index.md), and
[policies](../../../../delivered/advanced-delivery/policies/spec/index.md) epics have landed: `land` runs each
repository's policy a pass at a time, a pure action under `bzh:hub-action-purity`; every forge is spoken through the
async roles `ForgeRegistry` hands out — `IForgeRefs`, `IForgeChanges`, and `IForgeChecks`, with the operations the
policies epic added ([seam.md](../../../../delivered/advanced-delivery/policies/spec/seam.md)); and `ForgeBudgets` keys
each forge's rate budget by forge, secret, and bucket
([budget.md](../../../../delivered/advanced-delivery/policies/spec/budget.md)).

Paths are relative to `blizzard/src/blizzard/`.

The epic reaches beyond `blizzard`'s source. In `blizzard-context`, `domain/artifacts/delivery.md` names GitLab beside
GitHub as a forge a repository may land on; `verification/blizzard/e2e-scenarios/delivery-policy.md` gains the GitLab
scenarios; and blizzard's verifiability matrix declares the GitLab contract suite as its own method
([carry-over.md](./carry-over.md#the-contract-suite)). `docs/deployment/forges.md` describes declaring a GitLab forge,
the minimum version, and the policies a GitLab repository may choose. `blizzard-mock`'s forge gains a GitLab face
([carry-over.md](./carry-over.md#the-mock)).
