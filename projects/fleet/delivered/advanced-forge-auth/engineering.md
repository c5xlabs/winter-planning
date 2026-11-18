# Advanced forge authentication engineering

Engineering requirements for `epic:advanced-forge-auth`.

## What already holds

- The forge record and landing seam of `epic:advanced-delivery/forges`. This epic's configuration changes are
  coordinated with that epic's forge surfaces.
- The configured-record lifecycle, change log, and sealed secrets of `epic:live-config`, which the credential record
  joins as one more kind.
- A replaced secret takes effect at its next use: adapters are cached under a key carrying their records' and secret's
  revisions (`blizzard/src/blizzard/hub/live_config/__init__.py`).
- Landing policies' capability requirements are checked against `ForgeRegistry.offered` at configuration time and at
  each pass's start; `offered` already takes a repository and ignores it
  (`blizzard/src/blizzard/hub/forge/registry.py`).
- A requeued `land` reuses the change already open for its branch and skips repositories already landed
  (`blizzard/src/blizzard/hub/delivery/actions/landing/merge_on_checks.py`, `.../actions/land.py`).
- Unavailable, unreadable, refused, or empty checks never lead to a merge
  (`blizzard/src/blizzard/hub/delivery/actions/landing/checks.py`).
- `engineering:private-checks-proof` holds: a test App installed only on the private `paul-gross/blizzard-infra`, with
  Checks, Contents, and Pull requests read, minted an installation token that read a merged pull request's head and its
  check runs, matching the conclusions GitHub shows for it; a classic `repo` token matched too. No probe pull request
  and no merge.
- GitHub App installation authentication:
  <https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/authenticating-as-a-github-app-installation>
- GitHub App permissions for Checks:
  <https://docs.github.com/en/rest/authentication/permissions-required-for-github-apps#repository-permissions-for-checks>
- GitHub fine-grained PAT limitations:
  <https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#fine-grained-personal-access-tokens-limitations>

## Brief requirements

- engineering:no-credential-crosses-tenants — No credential, and no ability to use one, crosses the tenant boundary.
  Serves `product:credential-records`.
- engineering:secrets-out-of-evidence — Credentials and private response content never enter committed evidence or
  diagnostics. Serves `product:actionable-refusals`.

## Core and provider split (engineering:core-provider-split)

The core owns credential references, their lifecycle, tenant isolation, and a narrow boundary for obtaining usable
authentication. Each provider owns its configuration variants, permission vocabulary, token exchange, and request
authentication. GitLab and other forges fit without teaching delivery policies their protocols; the core holds no
universal OAuth engine and requires no GitHub field of another provider. Serves `product:credential-records`.

## One authentication boundary for every hub caller (engineering:shared-auth-boundary)

Delivery, commit verification, issue ingestion, and work-source annotations and closes use the selected credential
through their existing provider adapters. The provider acquires tokens and authenticates requests; actions, flows, and
landing policies receive no credential material. A long-lived client never holds an installation token past its usable
lifetime. Serves `product:credential-records`, `product:github-app-installations`.

## Managed token renewal (engineering:managed-token-renewal)

The hub renews an installation token before it expires, coordinates concurrent renewals of one authorization into one,
and recovers its tokens after a restart. Cached authorization is keyed so it never mixes tenants, forge servers,
installations, or access scopes. Serves `product:github-app-installations`.

## Rate and write accounting by authorization (engineering:accounting-by-authorization)

Write coordination and rate accounting follow a credential's effective authorization, so each renewed token is not
treated as an unrelated identity. Serves `product:github-app-installations`.

## Additive transition (engineering:additive-transition)

Every landing is additive to supported releases: existing PAT secrets, references, configuration documents, and clients
stay usable while the credential configuration arrives. Retiring superseded secret-reference fields, if needed, is a
separate cleanup outside this epic, once the line carrying the transition is cut. Serves `product:supplied-tokens`.

## Private-repository proof before App rollout (engineering:private-checks-proof)

Before the App binding is built out, a test App installed on an operator-approved private repository uses its
installation token to read a pull request's head and the exact check-runs endpoint delivery uses, and the expected
checks and conclusions are verified rather than a successful HTTP status. The proof opens no probe pull request and
performs no merge, and a public-repository success does not substitute for it. Serves
`product:github-app-installations`.
