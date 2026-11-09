---
initiative: lab
refinement: outline
epics:
  - bench
  - experiment
  - lab-runner
  - specimen
  - verdict
  - lab-board
  - seeded-defects
  - evidence
---

# Lab

The lab lets the people who tune the fleet run controlled experiments on it, so a change to a prompt, a model, a graph,
or a convention is judged on evidence rather than feel.

## The problem

`persona:harness-engineer` changes one thing, runs the fleet, and reads the results. Today that last step is mostly
faith. A plan-review prompt is rewritten, a reviewer moves from one model to another, a convention is added to the
harness every worker reads, and the evidence that it helped is a few nights that felt better than the week before. Those
nights ran different work on a moving codebase, with every node of every chunk free to vary at once, so a better plan, a
luckier build, and a stricter reviewer all arrive tangled in the same outcome. The engineer wants what any experimenter
wants: two runs that differ in exactly one respect, and enough of them to tell a real difference from the noise.

## The outcome

The lab gives them that, at the grain where the answer actually lives: the node-step, not the chunk. A whole chunk run
twice is a poor experiment. The plan diverges in the first hour, and everything downstream inherits the divergence, so a
question about reviewers is drowned by differences in plans the reviewers never chose. A lab instead holds everything
upstream of the question fixed, runs only the stretch of the graph the question is about, stops each run at the boundary
of that stretch, and compares what came out. It is cheaper by the same stroke: a trial costs one or two node-steps, not
a night.

The **bench** epic is the transformation at the heart of it: an experiment is a graph derived from the one under study,
keeping the node-steps in question, seeding everything upstream, and capping every exit. The **experiment** epic turns
hand-started trials into a measurement, a hub record that runs every arm against the same work in paired waves and stops
on an answer, a non-answer, or an empty budget. The **lab runner** epic brings the pinned, certified-clean workspace a
trial runs in, kept apart from the project workspaces real work uses. The **specimen** epic draws the work trials run
against from the fleet's own history, sealed from the future it already knows. The **verdict** epic reads the trials,
from the facts the fleet already records and from judges that are graphs themselves. The **lab board** epic is where an
experiment is read without burying the operator's board. The **seeded defects** epic measures what a gate misses by
planting what it should catch. And the **evidence** epic closes the loop, so a proposal to change the fleet carries an
experiment beside its argument and an adopted change keeps being watched.

## Stance

Nothing in this initiative is built yet. Before blizzard grows experiments, trials, and a board of its own, the lab has
to earn them by being useful without them. The first step happens outside blizzard entirely: a **winter lab**, a winter
workspace whose feature environments each hold a whole other workspace, pinned to one commit, with a lab extension that
tells the agents working there how to design an experiment, turn a graph node into a bench, run trials in paired waves,
and read the result. The harness engineer drives it by hand, at a terminal, on a few real questions — the reviewer that
seems to have gone soft, the convention that may or may not be pulling its weight, the cheaper model someone wants to
try on plan-review.

That manual lab is the proof this initiative waits on. If a handful of experiments run by hand change a decision someone
would otherwise have made by feel, the case for bringing the lab into the fleet makes itself, and these epics are built
knowing which parts of the work were worth automating and which were not. If they change nothing, the initiative has
cost a few afternoons instead of a quarter. Either way, what blizzard eventually builds is shaped by an experimenter's
actual habits rather than by this plan's guesses about them.

The manual lab needs more of winter than winter has today: a workspace hosted inside another workspace's environment,
with its own port band, the identity its parent hands it, its context sealed from the lab around it, and the whole
definition frozen to exact commits and fingerprinted so two copies are provably the same. Winter's nested-workspaces
work carries all of it, and each piece is useful to an ordinary workspace too. The lab can be run before that lands,
with a script doing what winter will later do, which is how its first experiments should be run.

Every epic in the lab holds to these, because a lab that gets them wrong produces confident nonsense:

- **Isolation is proven, never assumed.** A trial's session runs inside the lab's directory tree, and a harness that
  walks upward for instruction files will find the lab's own unless told otherwise. Before an experiment's first wave,
  the lab asks a trial session which instruction files it loaded and refuses to run if any lies outside the arm.
- **A trial that never reached its end state measured nothing.** A reviewer denied the command it needed reports that it
  could not review; scored as a miss, it reads as a reviewer that missed the bug. Such a trial is invalid, and the lab
  says why.
- **A bench runs under the fleet's own permissions.** Agents chain shell commands that no narrow allowlist anticipates,
  and a bench held to a tighter posture than the fleet stops measuring the node it was cut from. What guards an arm is a
  fingerprint taken before and after each trial; a trial that changed its own arm is void.
- **The control is run against itself.** Two identical arms disagree more often than intuition expects, and that
  disagreement is the ruler every other difference is read against.
- **The lab produces evidence, and a person decides what it is worth.** It does not choose configurations on the
  operator's behalf; routing live work across arms by their running results is a different product.
- **Its subject is always this fleet.** It is not a general benchmark harness for models: it studies this fleet, its
  graphs, and its project's own work, which is precisely the evidence no public benchmark can supply.
