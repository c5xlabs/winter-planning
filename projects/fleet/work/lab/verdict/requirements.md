# Verdict requirements

The requirements of `epic:lab/verdict`.

## Readings from recorded facts (product:fact-readings)

The verdict reads what the fleet already records about each trial: how many times a gate sent work back, how many
retries a node spent, whether the trial escalated or asked a question, the tokens and time each node-step took, and
which end state the trial reached.

Acceptance:

- Each trial shows its loops per gate, retries per node, escalations, questions, tokens and time per node-step, and end
  state.

## Judges are graphs (product:evaluation-graphs)

An evaluation graph receives what a trial's end state carried and judges it: the faceted reviews and garden axes the
fleet already holds itself to, run over each arm's output; a blind pairwise comparison that sees two outputs with the
arms hidden and the order swapped, judged by a model from a family neither arm used; and, where the question deserves
it, a person shown two results on the board and asked which they would merge. People's choices accumulate into the
measure the automated judges are later calibrated against.

Acceptance:

- A blind pairwise judge never sees which arm produced which output, and sees them in swapped order.
- A pairwise judge's model family is one neither arm used.
- A person's blind choices are kept as the measure judges are calibrated against.

## A verdict no agent wrote (product:computed-verdict)

The hub computes the verdict from the trials' facts; it is never a conclusion an agent wrote. It reports effect sizes
with their uncertainty rather than a winner, and says *inconclusive* when the evidence is.

Acceptance:

- Every verdict shows effect sizes with their uncertainty.
- A verdict can read inconclusive.

## Read against the control's own drift (product:control-drift)

Before an experiment's arms are compared to each other, the control is compared to itself, to measure how far two
identical runs drift apart, and a difference smaller than that drift is not reported as a difference.

Acceptance:

- Every verdict shows the control's drift beside the difference between arms.
- A difference smaller than the drift is not called a difference.

## Cost beside quality (product:cost-beside-quality)

Cost is reported beside quality rather than folded into it, because "nine-tenths as good at a third of the price" is a
decision the engineer should make with both numbers in view.

Acceptance:

- Every verdict shows each arm's cost and quality as separate numbers.

## Open questions

- Whether the arms of a harness comparison can ever be equal. Two harnesses differ in their tools and system prompts as
  well as their models, so "Claude against GPT" through two harnesses is really two bundles, and the lab should say so
  rather than report it as one variable.
- How judges are kept honest. A judge prefers longer answers, the first answer shown, and answers from its own family,
  and a fleet tuned against a judge learns to please it. Rotating judges and keeping people in the sample are the
  defenses; how much of each is the open part.
