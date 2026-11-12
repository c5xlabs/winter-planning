# Advanced forge authentication decisions

## Claim-time credential gate (decision:advanced-forge-auth/claim-time-gate)

**Decider:** product

**Open.** Whether claiming a chunk is refused when a repository its project lands on holds a credential whose runtime
offer cannot serve the repository's landing policy, so no worker starts on work that cannot land. The configuration and
pass-start gates cover every declared credential, and the hub-start warning names the rest. It blocks no requirement of
this epic; deciding yes adds an acceptance bullet to `product:readiness-check`.

## Fine-grained tokens and checks (decision:advanced-forge-auth/fine-grained-checks)

**Decider:** engineering

**Open.** Whether a fine-grained token is offered `checks.read`, decided by the private-repository proof's fine-grained
run: offered if it reads the check runs, withheld if refused. Until it is made, `FINE_GRAINED_WITHHELD` holds
`checks.read`, following GitHub's documented fine-grained limitations and the fleet's escalation on that read. It blocks
`product:supplied-tokens`, whose setup must say what a fine-grained token supports.

## Slice cut (decision:advanced-forge-auth/three-slices)

**Decider:** engineering

The epic is one epic, not a child epic per surface or adapter, cut into slices that each hold one mechanism, or a group
of thin entrypoints over one mechanism, so each is one reviewable change. References to secrets move to credentials by
expand and contract, whose contract step is the cleanup `engineering:additive-transition` leaves outside the epic.
Authentication moves by branch by abstraction, and its closing step, removing the direct secret reveal, lands with the
boundary itself, since nothing calls the old path once the boundary does.

## Installation tokens held in memory (decision:advanced-forge-auth/tokens-in-memory)

**Decider:** engineering

Minted installation tokens live only in the hub's memory and are minted again after a restart, because a mint costs one
call per hour per scope and storing them would add a sealed table whose loss or leak buys nothing.

## Readiness results not kept (decision:advanced-forge-auth/readiness-not-kept)

**Decider:** engineering

A readiness check answers and records counts in an event but stores no verdict, and no gate reads a past check. Gates
read only configuration and the token itself, which the adapter cache already invalidates, so nothing has to track when
a check went stale.

## Recovery through requeue (decision:advanced-forge-auth/recover-with-requeue)

**Decider:** engineering

An escalated delivery resumes with `blizzard hub chunk requeue` after the credential is fixed, rather than through a new
verb, because a requeued `land` already reuses the open change for its branch and skips repositories already landed.

## Credential carry-over (decision:advanced-forge-auth/credential-carry-over)

**Decider:** engineering

The migration creates one `github.token` credential per distinct forge and secret already named, shared by every
repository and work source naming that pair, with no variant. That keeps today's sharing of one token, budget, and write
lane, and a migration cannot reveal a token to classify it.

## One credential per authority (decision:advanced-forge-auth/one-credential-per-authority)

**Decider:** engineering

A forge holds at most one credential per authority, a token's secret or an App installation, and narrower access for
reading issues comes from the use's scope rather than a second credential. GitHub counts rate limits per authority, so
budgets and write lanes key on it, and two credentials for one authority would split one limit in two.

## No App registration record (decision:advanced-forge-auth/no-app-registration-record)

**Decider:** engineering

An App's id lives on each installation's credential and its key is a secret, with no record for the App itself. Two
installations of one App name the same key secret, so replacing the key is one secret write, and nothing in the epic
acts on an App apart from an installation.
