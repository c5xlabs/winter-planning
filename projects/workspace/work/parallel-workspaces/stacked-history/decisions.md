# Stacked history decisions

## Boundary derivation and cut scope (decision:parallel-workspaces/stacked-history/boundary-policy)

**Decider:** product owner and engineering

**Open.** Choose supported historical boundary evidence and explicit-cut scope when reflogs are missing or ancestry was
severed, including differing repo boundaries; this blocks finalizing `product:review-replay-boundaries`, not refusal
when a safe replay range cannot be established.

## Resume identity and lifetime (decision:parallel-workspaces/stacked-history/resume-policy)

**Decider:** product owner and engineering

**Open.** Choose whether continuation reconstructs intent or retains an operation identity, and how long completed and
cut evidence remains valid across edits or interruption; this blocks finalizing `product:resume-stack-rewrite` without
choosing a continuation file format here.
