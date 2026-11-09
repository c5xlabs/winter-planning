# GitLab decisions

## Rebase head gate (decision:advanced-delivery/gitlab/rebase-head-gate)

**Decider:** engineering

A rebased GitLab head is admitted on the hub's own word, that it requested this rebase, checked against what GitLab can
say about the result, rather than proven to carry the work unchanged as a GitHub base merge is. GitLab's comparisons
carry no blob ids, no per-file line counts, and no total that would show a commit list cut short, so the proof
`base_merge.py` gives a base merge is not available; the gate trades it for one the forge can answer, standing on the
narrow fact that the hub asked for this rebase and GitLab performed it.
