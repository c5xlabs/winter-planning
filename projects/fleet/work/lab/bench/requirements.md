# Bench requirements

The requirements of `epic:lab/bench`.

## Lab graph from a bench (product:lab-graph)

An engineer names a source graph, a bench, and a set of arms, and the lab mints a lab graph for each arm: the bench's
nodes, prompts, judgements, and the edges among them kept verbatim; everything upstream replaced by a seed, the
artifacts those nodes would have produced, supplied from a recorded run rather than produced again; and every edge
leaving the bench capped with an end state.

Acceptance:

- A lab graph runs the bench's nodes exactly as the source graph defines them.
- No node upstream or downstream of the bench runs in a trial.
- Any connected stretch of a graph, a single node included, can be named as a bench.

## End states (product:end-states)

An end state is a terminal that records which way the work was about to go and what it was carrying, then stops, named
for what would have happened next. It either returns the trial to the lab for analysis, or segues into an evaluation
graph whose nodes judge what the bench produced.

Acceptance:

- Each trial shows which end state it reached and what that end state carried.
- An end state may hand its trial to an evaluation graph.

## Arms are overlays (product:arm-overlays)

An arm names what differs from the source graph and nothing else: a declared session's model, effort, or harness, a
node's prompt or an edge's addendum, or a profile once `epic:worker-profiles` gives settings a name. An arm may also
name a workspace, the conventions, skills, agents, and tooling a worker finds around it when its session opens, which
are often the very thing under test and which the hub does not own. Every arm of an experiment is minted from the same
transformation, so two arms differ by exactly their overlays, and an arm with no overlay is the control.

Acceptance:

- Two arms' lab graphs differ only by their overlays.
- An arm with no overlay is shown as the control.

## Lab graphs stay apart from real work (product:lab-graphs-apart)

A lab graph is minted under a name of its own, so it never becomes the newest version of its source graph and no chunk
ever drifts onto it. Lab graphs never follow a newer mint, never land, and never turn the work items a bench node
proposes into real backlog; an end state is the end of a trial, not a delivery.

Acceptance:

- No live chunk ever runs on a lab graph.
- A trial lands nothing and adds nothing to the backlog.

## Refused at mint when it cannot be isolated (product:mint-refusals)

The lab refuses at mint a bench it cannot isolate, naming why before a single trial runs: a bench whose nodes read an
artifact nothing seeds, or a bench node that resumes a session begun upstream of it, as `pre-push` resumes the
conversation `build` started in `adv-dwf`. How such a bench is resolved is `decision:lab/bench/bench-sessions`.

Acceptance:

- Minting a bench with an unseeded input is refused, naming the input.
- Minting a bench with a node resuming an upstream session is refused, naming the node.

## Open questions

- How a seed reaches a bench. A node's inputs today arrive only with a judged step, so there is no way to place a chunk
  mid-graph with its upstream artifacts already present. Baking the seed into the lab graph as graph-scope artifacts
  works within the model as it stands; a first-class way to seed a chunk at a node may be worth having for its own sake.
