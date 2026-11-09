---
epic: test-suite/optimization
refinement: final
status: planned
---

# Optimization

Part of `initiative:test-suite`; the product requirements are below.

## The problem

Every chunk the fleet lands waits on the gate. The unit and component tier, already split into four parallel shards,
takes 11–17 min per shard, and a pull request waits on the slowest of them, about 17 min. Meanwhile the gate a pull
request passes does not build the board ahead of time, so a template error only the full build would catch first fails
on `master`.

## The outcome

A pull request waits less on the gate, without any assertion weakened, and the gate catches what today only `master`
catches.

## What this epic is not

The service tier's cost is not this epic's: the initiative's other epics replace a stack per test with a tenant per test
on a shared hub, under the initiative's guard that the tier never becomes the gate's slowest job, so nothing here
shortens a stack the initiative is retiring. Making tests implementation-independent belongs to the initiative's other
epics too, and where speed conflicts with it, the black-box line wins (`decision:test-suite/black-box-wins`).

## Requirements

### A shorter wait on the gate (product:shorter-gate)

A pull request's slowest required job finishes sooner, and every test still asserts what it asserted before.

Acceptance:

- The gate's critical path, the slowest required job on a `push` to `master`, is recorded against the baseline of 18
  October 2026.
- No test asserts less than it did before.

### The board built on every pull request (product:board-build-on-pr)

A template error in either board fails the pull request that introduced it, rather than the first push to `master` after
it.

Acceptance:

- A pull request with a template error the runner's board alone would catch fails its gate.
- The UI tests run on the board that build produced, so the board is built once per pull request.

### Releases still ship for arm64 (product:arm64-releases)

Building only amd64 on every push to `master` changes nothing for people running a release on arm64.

Acceptance:

- Every release is published for both amd64 and arm64.
