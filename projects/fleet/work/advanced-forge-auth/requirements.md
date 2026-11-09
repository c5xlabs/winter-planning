# Advanced forge authentication requirements

The requirements of `epic:advanced-forge-auth`.

## Authentication declared once, selected per use (product:credential-records)

A tenant manages typed credential records through the API, the CLI, the configuration document, and the board, with the
revision, change history, retirement, and secret handling other configured records have, and each repository and
forge-backed work source names a credential compatible with its forge. A record shows its kind and non-secret setup,
never a token or private key.

Acceptance:

- An operator creates, revises, retires, and reads the history of a credential from each of the four surfaces.
- A repository or work source refuses a credential that does not fit its forge.
- No surface shows a token or private key.
- Replacing key material or changing a binding takes effect at its next use, without a hub restart.

## PATs remain a straightforward option (product:supplied-tokens)

A PAT the operator supplies uses the existing secret store, and existing repository and work-source secret references
keep working without the operator re-entering a token. GitHub setup tells classic and fine-grained PATs apart, documents
the workflows and permissions each supports, and refuses known unsupported combinations rather than treating every PAT
as interchangeable.

Acceptance:

- An existing repository or work source keeps ingesting and landing through its current secret reference after upgrade.
- A classic PAT is a supported path for a private repository.
- A fine-grained PAT is not promised Checks API access GitHub does not support, and setup says so.

## Operator-owned GitHub Apps (product:github-app-installations)

An administrator creates an App on GitHub, installs it on the chosen account or organization and repositories, and
registers its App identity, installation, and private-key secret in their tenant; from then on the hub obtains, narrows,
and renews its access on its own. One App may serve many repositories, and no App is needed per project or user.

Acceptance:

- One registered App installation serves several repositories in a tenant.
- Work keeps flowing across token expiry and a hub restart, with no operator action.
- An expired token, a revoked or suspended installation, a removed repository, changed permissions, and a replaced key
  each produce a distinct, explained outcome.
- Setup and recovery instructions ship with the App binding.

## Readiness checked before expensive work (product:readiness-check)

Before work is promoted into a delivery path, an operator can run a read-only check that reports repository access and
any known permission or token-kind gap for the operations that use requires, exercising the real reads, including a
private repository's checks, rather than taking a successful repository read as proof of delivery access. What cannot be
proven without a write is stated as unproven.

Acceptance:

- The check never merges, changes a branch, or opens a probe pull request.
- A known incompatibility is reported before work enters the affected delivery path.
- A write permission the check cannot prove is reported as unproven, and the check does not claim a future change will
  satisfy repository policy.

## Refusals an operator can act on (product:actionable-refusals)

When a forge operation fails, the operator sees what was refused and what to change: the sanitized provider message,
HTTP status, operation or endpoint, repository, and request identifier when available. An invalid or expired credential,
insufficient access, an unsupported token capability, and a repository-policy refusal each read differently from a rate
limit or an outage.

Acceptance:

- No diagnostic shows a token, key, or authorization header.
- Checks that are unavailable, unreadable, or unexpectedly absent never lead to a merge.
- An operator replaces the credential, checks it, and resumes an already-escalated delivery without a duplicate pull
  request or lost landings.
- Insufficient permissions and lost access leave an in-flight delivery safe.
