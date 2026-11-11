---
name: work-guidance
depends-on:
  - context-command
---

# Publish the work-operation context branch

Deliver the work branch of `product:task-taxonomy` and the interactive work operations of `product:factory-operation`
through [Markdown sources](../spec.md#markdown-sources), using `engineering:context-example-proof` and the source and
navigation contracts established by `engineering:context-markdown-source` and `engineering:context-resource-boundary`.

## Change

- Add `work/index.md` and its `inspect.md`, `prepare.md`, `control.md`, and `respond.md` leaves; add the work branch to
  the authored root. Keep the index a router rather than an operator manual.
- Describe the existing inspection, ingestion, grouping, prerequisite, queue, promotion, chunk-control, question, and
  human-decision surfaces. Separate their authority and side effects from tenant-owned priority and approval policy.
- Reuse or rehome authoritative Markdown according to [packaging](../spec.md#packaging), keeping the task paths usable
  offline. Validate command examples against the shipped surface without executing mutations.
- Leave delivery-policy details, leased-worker interfaces, and runner-host administration to their own topic sources;
  link them only when the sources are present, as specified by the catalog's source contract.

## Acceptance

- Installed-CLI tests read `work` and `work/index` with identical results, then follow the authored requests to each of
  the four leaves; every page remains readable offline without a worker identity or source checkout.
- Coverage fixtures pin each work topic's command families and read-trigger routing against the source contract;
  preparation covers grouping before promotion and true prerequisites, control names scope and consequences, and
  response guidance distinguishes a question from a human decision.
- Example-surface validation accepts the documented command/option forms and rejects stale ones without invoking
  mutation bodies or contacting a real fleet. Existing operating facts have one Markdown owner.
- The built catalog remains valid with this branch added independently of execution guidance: no placeholders, missing
  links, or implicit dependence on a sibling slice's pages.
