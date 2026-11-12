---
epic: advanced-forge-auth
refinement: refined
status: planned
external: github:paul-gross/blizzard#2173
---

# Advanced forge authentication

A standalone epic; its requirements are in `requirements.md`.

## The problem

An operator who gives the fleet a private repository needs to know that it can finish the work it accepts. Opening a
pull request is not enough: the credential must also let the hub read its checks and merge it when the repository
permits. Today an incompatible credential can survive setup and fail only after an agent has finished, with an
explanation that says little more than “authentication failed.” For `persona:application-architect`, that is a night of
unattended work returned as a configuration problem they could have settled before it began.

## The outcome

A tenant makes an explicit choice of forge authentication, with GitHub PATs and operator-owned GitHub Apps as its first
bindings, and learns before any work starts whether the choice can carry that work to a merge. Repositories and work
sources select their authentication independently, and the rest of blizzard asks its forge to do the work without
knowing how it authenticated.

A forge identifies a server and its kind; a credential identifies how a particular use authenticates to that server. A
tenant may use several credentials on one forge, and several repositories and work sources may share one credential.
Reading issues need not carry the same privileges as landing changes, and neither a credential nor the ability to use it
crosses the tenant boundary.

What the operator has to understand is who maintains access, not which word appears in an HTTP header. With an
**externally supplied token**, such as a PAT, the operator supplies and replaces the secret, and blizzard does not
invent a way to renew it. With **provider-managed access**, the operator supplies an identity and its authorization, and
blizzard obtains and renews the access tokens that authorization permits. A GitHub App installation is one such
authorization. An OAuth grant is another, with a different setup and renewal protocol; a GitHub App installation is not
an OAuth grant, and a PAT may travel in the same header as a managed token.

## What this epic is not

- A GitLab or other forge authentication binding. The design admits one without a delivery rewrite; the provider's own
  integration supplies it when needed.
- An OAuth authorization flow or a generic grant framework without a concrete provider use case. Its eventual binding
  may reuse the managed-access lifecycle, not pretend that a GitHub installation is an OAuth grant.
- A Blizzard-owned public App shared across hosted customers, a central credential broker for independent hubs, or a
  shared App private key. Operator-owned Apps are sufficient.
- Creating Apps from inside blizzard, installation onboarding through GitHub manifests, or a marketplace listing.
  Administrators create and install their App on GitHub.
- Webhook registration, inbound event handling, or replacing polling. App authentication makes those possible but works
  without an inbound hub endpoint.
- Changing GitHub login, implementing CI/CD, or granting an App a bypass around branch protection and approval policy.
- Runner checkout and worker-side git credentials, which remain their own boundary.

## Depends on

- `epic:advanced-delivery/forges`, the forge record and landing seam this epic builds on. It does not wait for branch
  flows, landing policies, GitLab, or merge trains.
