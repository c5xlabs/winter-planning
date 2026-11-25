# Reproducible input decisions

## Reference resolution and lock meaning (decision:parallel-workspaces/reproducible-inputs/resolution-and-locks)

**Decider:** product owner and engineering

Resolution and lock meaning follow the reference tool, its quirks included, so a lock either tool writes means one thing
to both; [standalone-pins.md](./spec/standalone-pins.md) holds the rules. Converting a legacy format remains owned by
`decision:configuration-transition`.

## Update preservation (decision:parallel-workspaces/reproducible-inputs/update-preservation)

**Decider:** product owner and engineering

Re-pinning follows the reference tool, so a branch pin resets its local branch to `origin/<ref>` without a prompt and
uncommitted work is refused unless temporary preservation is requested; matching it keeps one behaviour across both
tools, at the cost of leaving local-only commits to the reflog.

## Content identity boundary (decision:parallel-workspaces/reproducible-inputs/fingerprint-boundary)

**Decider:** product owner and engineering

Definition identity follows the reference tool, digest included: tracked content of the workspace repository and of each
named standalone, so both tools give one digest for one definition; untracked files, component version, and content
edited during the computation are outside it.
