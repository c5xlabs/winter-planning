# CI feedback decisions

## How CI results are heard (decision:ci-feedback/ci-watch)

**Open.** What the CI seam watches in the reference stack: polling the forge's check runs, a webhook the hub exposes, or
both, and how a self-hosted hub without an inbound surface hears the news; it blocks `product:feedback-routed-home`.
