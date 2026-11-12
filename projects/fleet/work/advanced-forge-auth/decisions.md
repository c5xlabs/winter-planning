# Advanced forge authentication decisions

## Fine-grained tokens and checks (decision:advanced-forge-auth/fine-grained-checks)

**Decider:** engineering

A fine-grained token is not offered `checks.read`: GitHub lists the Checks API among fine-grained token limitations, and
the fleet's `forge-auth` escalation on a private repository came from exactly that read, and the operator has seen a
fine-grained token fail it. A fine-grained token that reads private check runs reopens this decision.

## Claim-time credential gate (decision:advanced-forge-auth/claim-time-gate)

**Decider:** product

Claiming a chunk is not refused for a credential that cannot serve its repository's landing policy. The configuration
gate, the hub-start warning, and the pass-start gate already catch the incompatibility, and a gate on the claim path
would add a token reveal to every claim for the one case they leave: an undeclared token whose warning was ignored.

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
