---
name: restack-plan
depends-on:
  - plan-reads
  - epic:parallel-workspaces/coordinated-git
---

# The restack plan

Adds `Chain`, `Refusal`, `PlanRefusal`, `Source`, `LinkRepo`, `Link`, `Plan`, and `plan` of
[restack-plan.md](../spec/restack-plan.md), Domain, Refusals, and Plan: the chain split into links top first, BASE and
`--cut` expanded for every non-pinned repository through `ref_tokens::expand` of
[ref-tokens.md](../../coordinated-git/spec/ref-tokens.md), which `epic:parallel-workspaces/coordinated-git` builds, then
every link read, refused all-or-nothing in the spec's order, or frozen with its boundaries and reversed bottom first.
Delivers the chain and refusal bullets of `product:declare-stack-order`, the boundary, safety, and cut bullets of
`product:review-replay-boundaries`, and `engineering:stack-boundary-integrity`, under
`decision:parallel-workspaces/stacked-history/boundary-policy` and
`decision:parallel-workspaces/stacked-history/token-expansion`, whose eager expansion is a registered deviation.

## Change

- `crates/core/src/restack.rs` gains the plan's types and `plan`: a repeat is the only refusal and reads nothing; a
  `RefTokenError` ends the run as its command error before any git call; then reads, missing refs, and per link the
  safety, boundary, and inversion refusals, every refusal listed.
- An unrefused plan holds every non-pinned repository per link in configuration order, a skipped one with boundary and
  source `None`; `--cut` is the bottom link's boundary only.
- `plan` writes nothing; no command calls it yet.

## Acceptance

- Unit tests over a recording program port: a repeated env, and `--cut` equal to BASE, are each the one `InvertedOrder`
  refusal, with the spec's link, `repos` and `element`, and no git call ran.
- Unit tests: BASE `{mian}` ends with `CommandError::RefToken` before any git call; BASE `{main}` with a non-pinned
  repository whose `main_branch` is empty refuses the same way, though that repository has none of the chain's branches.
- Component tests with real git: an env no repository has is `MissingRef` with `repos` `[]` and the env as `element`; a
  BASE no participating repository has is `MissingRef` with BASE as typed; a pinned project appears in no link.
- Component tests: a tracked change is `Dirty` and an untracked file is not; a stopped rebase is `RebaseInProgress`;
  HEAD detached or on another branch is `DetachedHead`; each names its repositories, sorted.
- Component tests: after the bottom predecessor is amended, the upper link's boundary is the predecessor's old tip with
  `ForkPoint`; a link with no fork point is `UnknownBoundary`, with no merge-base fallback.
- Component tests of `--cut`: a cut that is an ancestor of the env tip is `Cut`; a repository already replayed past the
  predecessor tip is `Cut`; neither is `CutNotAncestor`; a cut that does not resolve in a repository is `MissingRef`
  with `element` `"--cut"`, before `CutNotAncestor`; the upper links keep their fork points.
- Component tests: a two-link chain typed bottom first is `InvertedOrder` naming the repositories; a link whose env tip
  equals its boundary is not.
- Component tests: a same-named tag pointing elsewhere changes neither participation nor any boundary; a repository
  missing one link's env branch is skipped in that link alone.
- A component test: a chain with a dirty repository in one link and an unknown boundary in another lists both refusals,
  top link first, and no ref, index, or file changes.
- `cargo dev gate slice` exits 0.
