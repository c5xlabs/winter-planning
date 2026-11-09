# GitLab decisions

## Hourly budget notices (decision:advanced-delivery/gitlab/hourly-budget-notices)

**Decider:** engineering

A GitLab budget records `forge-budget-low` and `forge-rate-limited` each at most once an hour, rather than once per rate
window as a GitHub budget does, because GitLab's windows are a minute long and once per window would fill the event log
in a busy hour.

## No conditional reads (decision:advanced-delivery/gitlab/no-conditional-reads)

**Decider:** engineering

The GitLab adapter sends no conditional requests, unlike the GitHub adapter's `ETag` reads, because gitlab.com allows
about 2,000 authenticated requests a minute and the policies epic's cheap pass keeps a GitLab landing well inside that.

## Rebase head gate (decision:advanced-delivery/gitlab/rebase-head-gate)

**Decider:** engineering

A rebased GitLab head is admitted on the hub's own word, that it requested this rebase, checked against what GitLab can
say about the result, rather than proven to carry the work unchanged as a GitHub base merge is. GitLab's comparisons
carry no blob ids, no per-file line counts, and no total that would show a commit list cut short, so the proof
`base_merge.py` gives a base merge is not available; the gate trades it for one the forge can answer, standing on the
narrow fact that the hub asked for this rebase and GitLab performed it.
