# The mock forge

Paths are relative to `blizzard-mock/src/blizzard_mock/forge/`. engineering:mock-proven.

## Interfaces

### Routes

| Route                                                                                                       | Status                 | Answers                                                                                    |
| ----------------------------------------------------------------------------------------------------------- | ---------------------- | ------------------------------------------------------------------------------------------ |
| `POST /repos/{owner}/{repo}/git/refs`, `DELETE …/git/refs/heads/{branch}`                                   | Exists (`api/refs.py`) | Over the git backend                                                                       |
| `GET /repos/{owner}/{repo}/branches/{branch}`                                                               | New                    | `protected`, from the protection lever                                                     |
| `GET /repos/{owner}/{repo}/commits/{ref}/check-runs`                                                        | Changed                | Steerable runs on any revision; today every revision but a pull request's head reads green |
| `GET /repos/{owner}/{repo}/rules/branches/{base}`                                                           | New                    | The rules the protection, linear-history, and merge-queue levers set                       |
| `POST /graphql` `repository.mergeQueue(branch:)`                                                            | New                    | A queue when the merge-queue lever is set                                                  |
| `POST /graphql` `enqueuePullRequest`, `dequeuePullRequest`; `mergeQueueEntry`, `RemovedFromMergeQueueEvent` | New                    | A first-come-first-served merge queue proving each change with the steerable checks        |
| GitLab face: the `merge_trains` routes, the project's `merge_trains_enabled`, system notes for a removal    | New                    | Merge trains, beside the GitLab face in [gitlab's mock.md](../../gitlab/spec/mock.md)      |

### Levers

| Lever does                                                                                                | Proves                                                          |
| --------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| Fails a named check on every revision whose tree holds a given path                                       | A failing batch, and a split whose halves without the path pass |
| Runs checks only on branches under a given prefix                                                         | No checks on staging branches; `unproven`                       |
| Leaves a check queued forever                                                                             | `proof-timed-out`                                               |
| Turns the base's own checks red until cleared                                                             | The hold                                                        |
| Fails a named check once, passes it on its re-run                                                         | A flaky check ejects no one                                     |
| Sets protection, linear history, and a merge queue on a branch (feeding the rules and `mergeQueue` reads) | Probes; `advance-refused`                                       |
| Removes a queued change for a conflict, a failure, a timeout, or by hand                                  | Each `QueueRemoval`                                             |
| Answers a forge read with a 5xx                                                                           | A failed probe leaves the last answer                           |

## Open questions

- Lever names and parameters are not given; the mock's `LeverKind` (`domain/levers.py`) already has `branch_protected`,
  `ruleset`, `ref_checks`, and `base_checks_failed`, which some of these may extend rather than add. Engineering.
