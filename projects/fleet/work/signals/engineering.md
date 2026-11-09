# Inbound signals engineering

Engineering for `epic:signals`.

## What already holds

- `initiative:projects` gives a project no sources and no secrets of its own: a work source and the credentials it uses
  belong to the tenant, and a project only links them.

## Brief requirements

- engineering:push-binding — The signal endpoint is the work-source seam's first push binding; the seam has only ever
  pulled. Serves `product:signal-raises-item`.
- engineering:no-tenant-id-in-url — A push names its tenant and project through the endpoint's address or signing
  secret, never through a private tenant id in a public URL. Serves `product:project-endpoint`.

## Open questions

- **Engineering:** The signing scheme, and how a secret is rotated without a window where either the old or the new one
  is refused.
- **Engineering:** How a push names its tenant and project, since it carries no session and no runner credential.
