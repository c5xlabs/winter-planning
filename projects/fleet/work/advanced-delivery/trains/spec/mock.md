# The mock forge

Paths are relative to `blizzard-mock`.

## Interfaces

`blizzard-mock`'s forge grows what the train and queue proofs need, over its git backend:

- `POST` and `DELETE git/refs`, and `protected` on a branch read.
- Check runs on any revision steerable by its content: a lever that fails a named check on every revision whose tree
  holds a given path, so a batch carrying one bad chunk fails and the halves without it pass. Today every revision but a
  pull request's head reads green, which can prove neither a failing batch nor a split.
- Levers on those checks: one that runs them only on branches under a given prefix, so staging branches run none; one
  that leaves a check queued forever; one that turns the base's own checks red until cleared; and one that fails a named
  check once and passes it on its re-run.
- A branch-rules read (`rules/branches/{base}`) answering what the protection, linear-history, and merge-queue levers
  set, and GraphQL `repository.mergeQueue(branch:)` answering the merge-queue lever.
- A merge queue: GraphQL `enqueuePullRequest`, `mergeQueueEntry`, `RemovedFromMergeQueueEvent`, and
  `dequeuePullRequest`, ordered first come first served, proving each change with the steerable checks, with levers to
  remove a change for a conflict, a failure, a timeout, or by hand.
- On the GitLab face ([gitlab's mock.md](../../gitlab/spec/mock.md)), merge trains: the `merge_trains` routes, the
  project's `merge_trains_enabled`, and system notes for a removal.
- A lever that answers a forge read with a 5xx, so a probe can fail.
