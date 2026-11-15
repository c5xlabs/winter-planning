---
name: hub-fleet
depends-on:
  - hub-projects
  - hub-graphs
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
---

# Fleet registration and retirement services

Moves the fleet-registration row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:narrow-api-edge`; follows `decision:adopt-di/declared-cutover`.

## Change

- Register runner/host enrollment and report, retired-runner guard, detach, fleet and host-retirement services.
- Narrow fleet, fleet-host, host and runner handler dependencies to named services/read aliases.
- Retain canonical registry/service references for legacy execution; retire unused fields and bridges.

## Acceptance

- Enrollment/report/retirement and supported-runner API tests preserve version/refusal behavior and tenant isolation.
- Detach/retired guards have one owner across fleet and execution paths; retained bag fields share that instance.
- Strict typing, relevant component/API tests and graph/route gates pass with unchanged HTTP/wire models.
