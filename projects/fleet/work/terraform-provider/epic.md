---
epic: terraform-provider
refinement: outline
status: planned
---

# Terraform provider

A standalone epic; its requirements are inline below.

## The problem

A shop that adopts blizzard already describes its infrastructure somewhere, and for most shops that somewhere is
Terraform. The machine the hub runs on, its DNS, its backups — each is a resource in a plan someone reviews before it is
applied. Then the hub comes up, and everything that makes it *this* shop's hub — its tenants and who belongs to them,
its projects, the sources each project reads and the repositories it lands into, the credentials they use — is
configured with a different tool, in a different file, and reviewed in nobody's plan. Our own hosted hub lives exactly
this split: its host is Terraform in its infrastructure repository, and its contents are not.

## The outcome

`persona:application-architect` declares a hub's configuration where the rest of the estate already lives. A Terraform
provider for blizzard manages the hub's configured records as resources, so the plan that creates the host also states
what runs on it, and a change to a project's repositories is reviewed and applied like a change to a DNS record.

The provider is a client of the hub's configuration API and nothing more. `epic:live-config` built that API with this
provider named as its second consumer, and the guarantees it makes — stable identifiers, idempotent writes, a difference
shown before anything is written, retirement in place of deletion, and secrets that are written but never read — are
what keep the provider thin.

## Depends on

- `epic:live-config`, the configuration API the provider is a client of.
- `initiative:multi-tenancy` and `initiative:projects`, which give the hub the records worth managing. Built any
  earlier, the provider would be rewritten as each of them lands.
- `initiative:advanced-delivery` and `epic:advanced-deployment`, for the landing policies and deployment declarations
  the provider manages once those epics give them a home.

## Requirements

### A resource for every configured record (product:record-resources)

Every record an operator configures on the hub is a Terraform resource with a data source to read it: tenants and
memberships, projects, work sources, repositories, and secrets. Landing policies and deployment declarations join them
as the epics that own them give them a home.

Acceptance:

- An operator creates, changes, and reads each of these records from a Terraform plan.

### Destroy means retire (product:destroy-retires)

Removing a resource from a plan retires the record rather than erasing it, so a source's history stays with the chunks
that came from it and a mistaken destroy can be undone from the hub. What an apply does to a record still carrying live
work is `decision:terraform-provider/retiring-live-records`.

Acceptance:

- A record removed from a plan is retired on the hub, and its history remains.
- An operator restores a record a mistaken destroy retired, from the hub.

### Secrets that never reach state (product:write-only-secrets)

A secret is write-only: the provider sends it and the hub never returns it, so neither a plan nor a state file ever
carries a credential.

Acceptance:

- No plan output or state file holds a secret's value.

### Published where operators look (product:published-provider)

The provider is released to the public Terraform registry, works with OpenTofu, and is versioned against the hub's API,
so an operator can tell which provider a given hub accepts.

Acceptance:

- An operator installs the provider from the public Terraform registry, and from OpenTofu.
- An operator can tell from the provider's version which hub releases accept it.
