# Trains requirements

Requirements of `epic:advanced-delivery/trains`.

## Mechanical first (product:mechanical-first)

Inside a train, a base that moved costs a merge the forge makes, not a run of checks per chunk and never an agent. Only
a conflict with the base itself goes back to the chunk's graph as a conflict.

Acceptance:

- A chunk behind its base joins a batch without an agent being woken.
- A chunk that conflicts with the base is turned back to its graph as a conflict.

## Batches, proven once (product:batch-proof)

A flow gathers the submissions ready on its branch, combines them onto the base in the order they arrived, and proves
the combination with one run of the repository's checks. A batch that passes lands together. A batch that fails is split
and its halves proven apart until the chunk responsible is found, and every chunk the failure did not belong to lands
without it.

Acceptance:

- Several chunks ready on one branch land together after one run of the repository's checks.
- A batch with one bad chunk lands every other chunk, in the order they arrived, and turns back only the bad one.

## The combination is what lands (product:proven-tree-lands)

The branch advances to the very tree the flow proved, never to a tree assembled after the proof, so the revision that
lands is the revision that passed. A lone chunk that already holds its base and passed its own checks is advanced to
directly, under the same rule.

Acceptance:

- The revision on the branch after a batch lands is the revision whose checks passed.
- A lone chunk that holds its base and passed its own checks lands without a second run of checks.

## How big, how long (product:batch-settings)

A repository says how many chunks a batch may carry and how long a flow waits for a batch to fill before it proves what
it has, so a quiet repository lands a lone chunk promptly and a busy one amortises its checks.

Acceptance:

- A repository sets the most chunks a batch may carry and how long a flow waits for it to fill.
- On a quiet repository, a lone chunk lands once the wait runs out rather than waiting for company.
- On a busy repository, a batch is proven as soon as it is full.

## A train that knows when to stop (product:train-stops)

A proof that never finishes is abandoned after the same three hours a poll may run, and a base whose own checks are red
holds the train until it is green again, so one broken base costs one agent its repair rather than every chunk in line a
bounce.

Acceptance:

- A proof still unfinished after three hours is abandoned.
- While the base's checks are red, one chunk is turned back to repair it and the rest wait rather than bounce.
- The train moves again once the base is green.

## Capabilities probed per repository (product:probed-capabilities)

A forge's capabilities narrow to the repository: whether a merge queue or a merge train is in force, or whether the base
accepts a direct advance, is a fact the hub reads about one repository, not its whole forge, and a policy or a train
that needs one is checked against the repository.

Acceptance:

- Two repositories on the same forge can differ in whether a merge queue, a merge train, or a direct advance is
  available, and the hub tells them apart.
- A repository given a policy or train its forge offers but the repository does not have is refused with that reason.

## Handing the line to the forge (product:forge-queue)

A repository whose forge runs its own merge queue, GitHub's merge queue or GitLab's merge trains, may hand ordering to
it, as a fourth policy beside the three `epic:advanced-delivery/policies` ships. The flow submits each change to the
forge's queue and reports the forge's verdict, so a shop that already lives on its forge's train keeps it. What that
queue merges is the forge's promise, not the flow's.

Acceptance:

- A repository can choose the forge's queue as its landing policy, on GitHub and on GitLab.
- A conflict or failed checks in the forge's queue reach the chunk as its verdict.

## Ejecting what cannot join (product:ejection)

A chunk whose work conflicts with the base, or whose work the split found responsible for a failure, is turned back to
its own graph with that verdict, and the train moves on without it.

Acceptance:

- The flow's record says which chunk was ejected, from which batch, and why.

## The train, on the board (product:train-on-board)

A flow's lane shows the batch being proven, the chunks it carries, those ejected from it, and a train held for a red
base, so the operator watching a busy branch sees what is moving and what is not.

Acceptance:

- A batching flow's lane shows the batch in progress and the chunks it carries.
- A lane shows the chunks recently ejected from it.
- A lane held for a red base says so.
