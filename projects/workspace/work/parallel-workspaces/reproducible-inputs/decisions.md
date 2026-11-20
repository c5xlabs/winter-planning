# Reproducible input decisions

## Reference resolution and lock meaning (decision:parallel-workspaces/reproducible-inputs/resolution-and-locks)

**Decider:** product owner and engineering

Resolution and lock meaning follow the reference tool. A `ref` resolves against on-disk refs in the order `origin/<ref>`
branch, then tag, then commit, and the first match wins without reporting a shadowed candidate; nothing falls back to
the default branch. A lock entry is fresh when its `ref` equals the declared `ref`. A fresh tag or commit entry is
materialized at its recorded commit; a fresh branch entry lands on the local branch tracking `origin/<ref>` as last
fetched, not on the recorded commit. A missing or stale entry is re-resolved, refused only when the checkout has
uncommitted changes, untracked files included. An unreadable, malformed, or unsupported lock is the configuration error
of the delivered lock format. A recorded commit absent from the object store surfaces as that checkout's failure.
Fetching keeps a local tag whose remote tag moved, so a moved tag resolves where it was. An annotated tag records its
tag object's id, which never equals the checked-out commit, so it never reads as up to date and always reads as drifted.
On a pull a branch pin integrates in the pull's mode, and its entry is rewritten only after a fast-forward. Converting a
legacy format remains owned by `decision:configuration-transition`.

## Update preservation (decision:parallel-workspaces/reproducible-inputs/update-preservation)

**Decider:** product owner and engineering

Re-pinning follows the reference tool. A branch pin is checked out by resetting its local branch to track
`origin/<ref>`, so local-only commits on that branch are left reachable only through the reflog, without a prompt; a tag
or commit pin is checked out detached. Uncommitted changes, untracked files included, refuse the re-pin unless the
caller requests temporary preservation, which stashes tracked changes before the checkout and pops after it on every
exit path. A failed pop leaves the stash in place and reports that it needs manual resolution. A tree dirty only by
untracked files stashes nothing, and the pop then takes whatever stash is on top.

## Content identity boundary (decision:parallel-workspaces/reproducible-inputs/fingerprint-boundary)

**Decider:** product owner and engineering

Definition identity follows the reference tool, digest included. It covers the workspace repository's tracked content
and each standalone's name and tracked content: a clean repository contributes its `HEAD` tree, and one with tracked
changes contributes the tree of its index brought up to the working tree. Component version and untracked local
configuration are outside it, a submodule contributes only its recorded commit, and checkout filters apply as Git's
staging applies them. The digest is SHA-256 over the reference's canonical JSON preimage at digest version 1, so both
tools give one digest for one definition. Unreferenced objects may be written to the object store; no consistency is
claimed for content edited during the computation.
