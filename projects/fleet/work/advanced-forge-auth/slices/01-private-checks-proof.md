---
name: private-checks-proof
---

# Private-checks proof

Builds the proof tool and runs it against an operator-approved private repository, before any App code. Delivers
`engineering:private-checks-proof`. Builds the proof tool in [authorization.md](../spec/authorization.md#interfaces).
Its fine-grained run confirms `decision:advanced-forge-auth/fine-grained-checks`.

## Change

- `blizzard/tools/proofs/github_private_checks.py`.
- Leaves alone: everything the hub runs; the tool is not imported by it.

## Acceptance

- A unit test over a recorded transport shows the tool calls only the pull request read, the check-run read, and the
  mint, and prints no token, response body, or check name.
- Run against the operator-approved private repository, the tool prints `app pull=200 check-runs=200 matched=yes`.
- The run's fine-grained line shows the check-run read refused, confirming
  `decision:advanced-forge-auth/fine-grained-checks`; a `200` there reopens the decision before any later slice lands.
