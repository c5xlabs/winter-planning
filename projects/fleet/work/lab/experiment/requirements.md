# Experiment requirements

The requirements of `epic:lab/experiment`.

## The experiment record (product:experiment-record)

An experiment names a hypothesis, the bench and the lab graph minted for each arm, the specimens to run them against,
how many times to repeat each pairing, the evaluation each trial's end state feeds, the budget it stops itself at, and
how many of its trials may run at once. It is launched deliberately, the way a routine's run is, because it spends money
the moment it starts.

Acceptance:

- An engineer defines each of these on an experiment before launching it.
- Nothing runs until the engineer launches the experiment.
- No more of its trials run at once than it declares.

## Trials are chunks that wrap no work (product:trial-chunks)

Each trial is an ordinary chunk carrying its experiment, arm, specimen, and replicate. It wraps no backlog item, so it
can never close one, never collide with the live chunk working the same item, and never be mistaken for work that owes a
landing. What a chunk that wraps nothing is, is `decision:lab/experiment/trial-work-items`.

Acceptance:

- A trial shows its experiment, arm, specimen, and replicate.
- A trial closes no work item and blocks no live chunk.
- A trial survives a runner crash the way any chunk does.

## Paired waves (product:paired-waves)

The experiment mints its trials in waves, each wave holding every arm of the same specimen together, so the arms run
side by side through the same hour of provider weather and against the same state of the fleet. As trials reach their
end states it mints whatever the comparison needs next: a judging chunk that receives two trials' outputs once both
exist, or a person's blind choice on the lab board.

Acceptance:

- Every arm of a specimen runs in the same wave.
- A comparison of two trials starts only once both have reached their end states.

## Stopping on an answer (product:stopping-rule)

Between waves the experiment reads what it has, and stops minting when the difference between the arms is clear, when it
is clearly smaller than the drift between two identical runs, or when the budget is spent, and says which of the three
ended it. Its stopping rule is fixed when it launches, never chosen after its numbers are in, since stopping early on a
lucky streak is the oldest mistake in experimentation.

Acceptance:

- A finished experiment says whether a clear answer, a clear non-answer, or the budget ended it.
- An experiment's stopping rule cannot change after launch.
- An experiment mints no trial once its budget is spent.

## Repetitions spent where they inform (product:repetition-spread)

Twenty repetitions of a single specimen say a great deal about that one piece of work and nothing about the next; ten
specimens run twice each say much more about the fleet. The experiment spends enough repetitions of the control against
itself to know how noisy a single trial is, and spreads the rest across specimens.

Acceptance:

- Each experiment runs the control against itself.

## Trials yield to real work (product:trial-urgency)

Trials travel the same ready queue as everything else, at an urgency the experiment names, the way a routine names the
urgency its work enters at, falling back to the tenant's default the same way when that urgency is retired. An engineer
who keeps a low group under `epic:urgency`, a `background` below the work that matters, puts trials there so they yield
to real work. An arm that names a harness reaches only the lab runners that offer it, by the same harness choice every
declared session already makes.

Acceptance:

- Trials at a lower urgency than live work wait while that work is ready.
- A trial whose arm names a harness runs only on a runner offering it.

## Open questions

- When the conductor stops: the statistical rule that decides a difference is clear, or clearly smaller than the drift.
