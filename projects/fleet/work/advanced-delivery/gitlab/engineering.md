# GitLab engineering requirements

Engineering requirements of `epic:advanced-delivery/gitlab`.

## What already holds

This epic adds a second adapter behind the landing seam and changes no policy. It builds on the delivered
`epic:advanced-delivery/actions`, `epic:advanced-delivery/forges`, `epic:advanced-delivery/flows`, and
`epic:advanced-delivery/policies`:

- `land` runs each repository's policy a pass at a time, a pure action under `bzh:hub-action-purity`
  ([actions spec](../../../delivered/advanced-delivery/actions/spec/index.md)).
- The forge record, the seam's roles, its vocabulary, and its retry rules are the forges epic's
  ([forges spec](../../../delivered/advanced-delivery/forges/spec/index.md)). Every forge is spoken through the async
  roles `ForgeRegistry` hands out: `IForgeRefs`, `IForgeChanges`, and `IForgeChecks`.
- The flow and its submissions are the flows epic's
  ([flows spec](../../../delivered/advanced-delivery/flows/spec/index.md)).
- The policies, the seam operations they added ([seam.md](../../../delivered/advanced-delivery/policies/spec/seam.md)),
  the order a project lands its repositories in, the combining step, and the rate budget are the policies epic's
  ([policies spec](../../../delivered/advanced-delivery/policies/spec/index.md)). `ForgeBudgets` keys each forge's rate
  budget by forge, secret, and bucket ([budget.md](../../../delivered/advanced-delivery/policies/spec/budget.md)).
- What each policy proves holds on GitLab
  ([policies' landing.md](../../../delivered/advanced-delivery/policies/spec/landing.md#what-each-policy-proves)). A
  GitLab project merging fast-forward or semi-linear refuses a merge request not up to date with its base, so there
  `merge-on-checks` merges only a change whose checks ran on the base it lands on; the forge closes the gap a
  merge-commit project leaves open. A project running merged-results pipelines proves the merge GitLab builds rather
  than the head, with no change to what the hub does.

## GitHub landing unchanged (engineering:github-unchanged)

An installation that predates the epic lands as it did. The epic adds no table and no column, so it carries no
migration, and a GitHub origin keys exactly as before ([namespaces.md](./spec/namespaces.md)), so no stored pointer or
marker written before the epic changes meaning.

Acceptance:

- A fixture store captured before the epic carries over, and every GitHub chunk lands with the outcomes and markers it
  would have before.
- A GitHub origin's pointer key is the one it had before the epic.

## Runner and hub in one release (engineering:runner-hub-release)

`repo_identity` and `pointer_key` change on the runner as well as at the hub, so the epic ships the runner and hub in
one release. A runner a release behind its hub keeps working for every GitHub repository; only a GitLab repository in a
nested group needs both sides at this release. Serves `product:nested-groups`.

## GitLab contract suite (engineering:gitlab-contract-suite)

A contract suite runs the GitLab adapter against a real GitLab CE container, pinned at the supported minimum, and is
owed by any change to the GitLab adapter. The mock forge's GitLab face ([mock.md](./spec/mock.md)) is written from
GitLab's documentation, and the behaviours below are the ones it would get wrong silently. Booting GitLab takes minutes
and needs docker, so the suite is a local-only `mise run gitlab-contract`, as the image smoke is
(`blizzard-context:/verification/blizzard/commands/packaging.md`), declared as its own method in blizzard's
verifiability matrix.

Acceptance:

- The suite covers every adapter operation, the `locked` state, a retried job, a merged-results pipeline, and a rebase.
- The suite runs the GitLab scenarios of `engineering:gitlab-scenarios`.

## GitLab scenarios (engineering:gitlab-scenarios)

Every GitLab behaviour the epic promises is proven end to end against the mock forge's GitLab face, and the same
scenarios run in the contract suite. Serves `epic:advanced-delivery/gitlab`.

Acceptance:

- A GitLab forge is declared, a repository on it created, and `fast-forward` refused on it naming `refs.advance`; a
  server below 15.6 is refused `forge_version_unsupported` at its first call and the visit escalated `forge-config`.
- A chunk lands into a GitLab repository by `merge-on-checks` and by `park`; a change passing through `locked` lands
  rather than failing; a retried job that passed reads green; a merged-results pipeline is read and waited on; a
  person's merge of a parked merge request is found within two minutes.
- A rebase the hub requested passes the head gate; one that changed the set of paths, one nobody requested, and a
  person's push after the rebase do not. The crash point between the marker and the request is reached by the kill-9
  sweep, and the next pass requests the rebase again.
- Two projects named alike in two groups on one server are two repositories, each landing its own chunk.
- A chunk spanning a GitHub repository and a GitLab one lands both in its project's order; with the GitLab repository
  parked, the GitHub one ranked after it waits for the person's merge.
- With the GitLab face's rate lever lowering the remaining, a GitLab key slows rather than fails, and
  `forge-rate-limited` is recorded at most once an hour for it.
- Every invariant the earlier epics declared passes after every scenario above.

## GitLab documentation (engineering:gitlab-docs)

The epic reaches beyond `blizzard`'s source. In `blizzard-context`, `domain/artifacts/delivery.md` names GitLab beside
GitHub as a forge a repository may land on, and `verification/blizzard/e2e-scenarios/delivery-policy.md` gains the
GitLab scenarios. `docs/deployment/forges.md` describes declaring a GitLab forge, the minimum version, and the policies
a GitLab repository may choose. Serves `product:gitlab-forge`.
