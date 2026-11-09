# Tenant telemetry engineering

Engineering for `epic:tenant-telemetry`.

## What already holds

- `initiative:multi-tenancy` names the tenant on every request span and every exported fact row, and keeps trace export
  and fact egress hub-wide.
- The hub's sweep spans are tenant-less.
- The fleet-spans, platform-spans, and runner-spans epics of `initiative:tracing` define what a span is.

## Brief requirements

- engineering:route-by-tenant-attribute — The tenant attribute that tags hub-wide export today routes each span and
  egress file to its tenant's destination. Serves `product:no-cross-tenant-telemetry`.

## Open questions

- **Engineering:** Whether runners, which export their own spans locally, forward per tenant too, or only the hub's
  spans are routed.
