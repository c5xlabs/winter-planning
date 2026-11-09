# Inbound signals decisions

## What an endpoint belongs to (decision:signals/endpoint-owner)

**Open.** An endpoint "per project" has to keep its signing secret on some tenant record, since a project has no secrets
of its own: the source it raises items in, the project's link to that source, or a signal record of its own; the rate
bound is already stated per source. It blocks `product:project-endpoint`.
