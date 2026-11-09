---
epic: tenant-telemetry
refinement: outline
status: planned
---

# Tenant telemetry

A standalone epic; its requirements are inline below.

## The problem

A hub that hosts several tenants narrates every one of them to a single place. Its traces go to the observability
backend whoever runs the hub chose, each span tagged with the tenant it belongs to, and that suits the person who runs
the hub. It does not suit the tenant. The team whose fleet lives in one tenant already has an observability backend of
its own, and the story of its nights arrives somewhere it cannot see, mixed with other tenants' stories it should never
be able to read.

## The outcome

Each tenant names where its own telemetry goes. The hub keeps its hub-wide export for the person who operates it, and
beside it forwards each tenant's traces, and, where `initiative:fact-egress` writes its files, each tenant's facts, to
the destination that tenant declared, with that tenant's credentials, and nothing of any other tenant's. A tenant that
declares nothing is narrated only to the hub's own destination, as today.

## Depends on

- `initiative:multi-tenancy`, which names the tenant on what the hub exports; this epic is the step past it.
- `initiative:tracing`, whose fleet-spans, platform-spans, and runner-spans epics define what a span is.

## Requirements

### A destination per tenant (product:tenant-destination)

A tenant declares an OpenTelemetry endpoint, its protocol and headers, and a credential from the tenant's secret store,
configured the way `epic:live-config` configures every other record.

Acceptance:

- A tenant that declares a destination receives its telemetry there.
- A tenant that declares nothing receives nothing, and its telemetry still goes to the hub's own destination.

### Forwarding without leaking (product:no-cross-tenant-telemetry)

Every span, and every egress file, reaches only the tenant it belongs to.

Acceptance:

- A tenant's destination never receives a span or egress file belonging to another tenant.

### Only a tenant's own story (product:own-story-only)

A tenant receives the spans of its own requests and chunks and the fact rows that name it. The hub's sweep spans are the
hub's own: they carry no tenant, and no tenant ever receives them.

Acceptance:

- No tenant's destination receives a hub sweep span.

### A slow backend costs only its own tenant (product:isolated-backends)

One tenant's unreachable collector never delays another tenant's export or the hub's own.

Acceptance:

- While one tenant's collector is down, every other tenant's export and the hub's own are not delayed.

## Open questions

- **Product owner:** Whether a tenant's destination receives the platform spans of the requests it made, or only the
  spans of its own chunks. Sweep spans are settled: they never go to a tenant.
- **Product owner:** Whether per-tenant fact egress belongs here or in `initiative:fact-egress`.
