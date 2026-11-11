# Stacked history engineering requirements

For `epic:parallel-workspaces/stacked-history`.

## Separate historical evidence from moving bases (engineering:stack-boundary-integrity)

All required replay boundaries are established before the first mutation; execution resolves each new base after lower
links complete. Local branch identity cannot be satisfied by a tag, including at evidence and execution reads. Serves
`product:review-replay-boundaries` and `product:replay-dependent-work`.

## Verify replay preservation against real Git (engineering:stack-preservation-evidence)

Disposable real-Git coverage includes amended predecessors, squash/severed ancestry, same-named tags, empty replay
ranges, multi-repository conflicts, and resumed cuts. Checks compare intended replay content and commit subjects, not
just old SHA reachability or an outcome enum. Serves `product:replay-dependent-work` and `product:resume-stack-rewrite`.

## Paused-state evidence is mandatory (engineering:stack-stop-classification)

A failed Git invocation is classified as a conflict pause only when the worktree's actual rebase state supports that
classification; other failures retain their cause and prior completions. Interruptions and concurrent writers cannot
silently discard continuation evidence. Serves `product:resume-stack-rewrite`.
